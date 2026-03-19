# GitHub Actions ワークフロー

このディレクトリには、GitHub Projectの運用を自動化するためのReusable workflowが含まれています。各リポジトリからはCaller workflowを通じてこれらを呼び出します。

## GitHub Actionsとは

GitHub Actionsは、リポジトリ上で発生するイベント（IssueのClose、PRのマージなど）をトリガーにして、あらかじめ定義した処理を自動実行する仕組みです。ワークフローは `.github/workflows/` 内のYAMLファイルで定義されています。

実行状況やログは、各リポジトリの Actions タブから確認できます。

## Iteration自動付与 (`set-iteration-on-close.yml`)

**目的**: IssueやPRがCloseされたとき、そのIssue/PRが紐付いているGitHub Projectに「現在のIteration」を自動で設定します。これにより、システム系などが「いつ何が完了したか」をProject上で把握できるようになります。

**動作の流れ**:

1. IssueまたはPRがCloseされる
2. GitHub Actionsが自動で起動する
3. そのIssue/PRが紐付いている全てのGitHub Projectを確認する
4. Iterationフィールドを持つProjectに対して、現在の日付に対応するIterationを自動設定する
5. Iterationフィールドがない、または現在のIterationが見つからないProjectはスキップされる

### 他のリポジトリへの導入方法

以下の内容で `.github/workflows/close-set-iteration.yml` を作成するだけで導入できます。

```yaml
name: Set Iteration on Close

on:
  issues:
    types: [closed]
  pull_request:
    types: [closed]

jobs:
  set-iteration:
    uses: ut-issl/.github/.github/workflows/set-iteration-on-close.yml@main
    secrets:
      ITERATION_AUTOMATION_APP_ID: ${{ secrets.ITERATION_AUTOMATION_APP_ID }}
      ITERATION_AUTOMATION_APP_PRIVATE_KEY: ${{ secrets.ITERATION_AUTOMATION_APP_PRIVATE_KEY }}
```

> [!NOTE]
> この自動化は、Organization Secretsに登録されたGitHub App（`ITERATION_AUTOMATION_APP_ID`, `ITERATION_AUTOMATION_APP_PRIVATE_KEY`）の認証情報を利用しています。新たにリポジトリを追加する場合は、Organization Secretsの「Repository access」に対象リポジトリを追加する必要があります。

## 系ラベル自動付与 (`auto-label-subsystem.yml`)

**目的**: IssueやPRが作成されたとき、リポジトリに設定されている **Topic** をもとに、対応する系ラベル（`sys:CDH` など）を自動で付与します。

**動作の流れ**:

1. IssueまたはPRが作成される
2. リポジトリのTopicを取得する
3. Topic名からサフィックスを抽出し、マッピングテーブルに基づいて系ラベルを特定する
4. 該当するラベルをIssue/PRに付与する

**Topicと系ラベルのマッピング**:

| Topicサフィックス | ラベル | 例 |
|---|---|---|
| `cdh` | `sys:CDH` | `geox-cdh` |
| `thermal` | `sys:熱` | `geox-thermal` |
| `structure` | `sys:構造` | `geox-structure` |
| `comm` | `sys:通信` | `geox-comm` |
| `aocs` | `sys:姿勢` | `geox-aocs` |
| `power` | `sys:電源` | `geox-power` |
| `orbit` | `sys:軌道` | `geox-orbit` |
| `propulsion` | `sys:推進` | `geox-propulsion` |
| `ground` | `sys:地上` | `geox-ground` |
| `payload` | `sys:PL` | `geox-payload` |
| `elec` | `sys:電装` | `geox-elec` |
| `system` | `sys:システム` | `geox-system` |
| `sw-team`（完全一致） | `sys:SW` | `sw-team` |
| `se-team`（完全一致） | `sys:SE` | `se-team` |

### 他のリポジトリへの導入方法

以下の内容で `.github/workflows/auto-label-subsystem.yml` を作成するだけで導入できます。

```yaml
name: Auto Label Subsystem

on:
  issues:
    types: [opened]
  pull_request:
    types: [opened]

jobs:
  auto-label:
    uses: ut-issl/.github/.github/workflows/auto-label-subsystem.yml@main
```

> [!NOTE]
> この自動化は `GITHUB_TOKEN` のみで動作し、追加のSecrets設定は不要です。リポジトリのTopicは GitHub の Settings > General > Topics から設定できます。Topic名は [issl-autoproject の rules.yaml](https://github.com/ut-issl/issl-autoproject/blob/main/rules.yaml) と同じ命名規則に従ってください。
