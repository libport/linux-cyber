# *Introduction to Agile Development and Scrum* Final Project
In this scenario, you will play the roles of a product owner, scrum master, and developer. As a product owner, you will create stories for your team and organize these stories into a product backlog. As a scrum master, you will create a sprint milestone and ensure that a subset of the stories is ready to be placed in a sprint plan. As a developer, you will create the sprint backlog and execute some of the stories by moving them across the kanban board in a simulated sprint.

Your team has been tasked with developing the back-end product catalog for an e-commerce website. Stakeholders require the ability to create, retrieve, update, and delete products from the catalog, along with features like indicating product likes and hosting on a cloud environment with automated deployments.

Your team will use a kanban board to create a backlog and sprint plan for this work. As the product owner, you will drive the process by leveraging the skills learned in the lessons and labs to create a new GitHub repository and kanban board and fill the kanban board with issues that will become user stories.
## Stakeholder requirements
Following are the requirements from your stakeholders that you should use to create the user stories in Kanban board.

1. Need the ability to create a product in the catalog.
2. Need the ability to retrieve a product from the catalog.
3. Need the ability to update a product in the catalog.
4. Need the ability to delete a product from the catalog.
5. Need the ability to **Like** a product in the catalog.
6. Need the ability to **Dislike** a product in the catalog.
7. Need the ability to list all products in the catalog.
8. Need the ability to query a subset of products in the catalog.
9. Must be hosted in the cloud.
10. Must have automation to deploy new changes to the cloud.
## Tasks
1. Create a new GitHub repository called `agile-final-project` and ensure it is public.
2. Create a **Project** in your GitHub repository and name it **Final Project**.
3. Create an issue template for the new repository similar to the labs.
4. Create issues for the stakeholder requirements listed above (10 in total). Fill in the **As a… I need… So that…** section of the template.
5. Move requirements 7 and 8 issues into the **Icebox**.
6. Move the remaining issues into the **Product Backlog**.
7. Conduct a **Sprint Planning** meeting. Add the top 4  stories to the sprint, assign them to the sprint milestone, and assign  story point estimates. Move these 4 stories to the **Sprint Backlog**.
##  Solution
### Authenticate and create the repository
```bash
set -euo pipefail

gh --version

if ! gh auth status >/dev/null 2>&1; then
  gh auth login --web --git-protocol https
fi

gh auth refresh -h github.com -s project
gh auth setup-git

OWNER="$(gh api user --jq '.login')"
REPO="agile-final-project"
FULL_REPO="$OWNER/$REPO"

gh repo create "$FULL_REPO" \
  --public \
  --clone \
  --add-readme \
  --description "Agile planning project for an e-commerce product catalog"

cd "$REPO"
```
### Add and commit the user-story template
````bash
mkdir -p .github/ISSUE_TEMPLATE

cat > .github/ISSUE_TEMPLATE/user-story.md <<'MARKDOWN'
---
name: User Story
about: This template is for creating user stories.
title: ''
labels: ''
assignees: ''
---

**As a** [role]

**I need** [function]

**So that** [benefit]

### Details and Assumptions

- [document what you know]

### Acceptance Criteria

```gherkin
Given [some context]
When [certain action is taken]
Then [the outcome of action is observed]
```
MARKDOWN

cp .github/ISSUE_TEMPLATE/user-story.md user-story.md

git config user.name >/dev/null 2>&1 || git config user.name "$OWNER"
git config user.email >/dev/null 2>&1 ||
  git config user.email "${OWNER}@users.noreply.github.com"

git add .github/ISSUE_TEMPLATE/user-story.md user-story.md
git commit -m "Add user story issue template"
git push
````
### Create and configure the public Project
```bash
PROJECT_NUMBER="$(
  gh project create \
    --owner "$OWNER" \
    --title "Final Project" \
    --format json \
    --jq '.number'
)"

gh project edit "$PROJECT_NUMBER" \
  --owner "$OWNER" \
  --visibility PUBLIC \
  --description "Product backlog and sprint plan for the product catalog"

gh project link "$PROJECT_NUMBER" \
  --owner "$OWNER" \
  --repo "$REPO"

gh project field-create "$PROJECT_NUMBER" \
  --owner "$OWNER" \
  --name "Workflow" \
  --data-type SINGLE_SELECT \
  --single-select-options \
  "Icebox,Product Backlog,Sprint Backlog,In Progress,Review,Done"

gh project field-create "$PROJECT_NUMBER" \
  --owner "$OWNER" \
  --name "Story points" \
  --data-type NUMBER
```
### Create user stories
```bash
create_story() {
  local title="$1"
  local role="$2"
  local need="$3"
  local benefit="$4"
  local assumption="$5"
  local given="$6"
  local when="$7"
  local then="$8"
  local body

  body="$(printf '%s\n' \
    "**As a** ${role}" \
    "" \
    "**I need** ${need}" \
    "" \
    "**So that** ${benefit}" \
    "" \
    "### Details and Assumptions" \
    "" \
    "- ${assumption}" \
    "" \
    "### Acceptance Criteria" \
    "" \
    '```gherkin' \
    "Given ${given}" \
    "When ${when}" \
    "Then ${then}" \
    '```')"

  gh issue create \
    --repo "$FULL_REPO" \
    --title "$title" \
    --body "$body"
}

ISSUE_URLS=()

add_story() {
  ISSUE_URLS+=("$(create_story "$@")")
}

add_story \
  "Create a product in the catalog" \
  "catalog administrator" \
  "the ability to create a product in the catalog" \
  "new products can be made available to customers" \
  "A product requires a name, description, price, and inventory count." \
  "I provide valid details for a new product" \
  "I submit a request to create it" \
  "the catalog stores the product and returns it with a unique ID"

add_story \
  "Retrieve a product from the catalog" \
  "customer" \
  "the ability to retrieve a product from the catalog" \
  "I can review its details before making a purchase" \
  "Products are retrieved using their unique IDs; unknown IDs return not found." \
  "a product exists in the catalog" \
  "I request the product using its ID" \
  "the catalog returns the complete product details"

add_story \
  "Update a product in the catalog" \
  "catalog administrator" \
  "the ability to update a product in the catalog" \
  "product information remains accurate" \
  "Updates are validated and do not change the product's unique ID." \
  "a product exists and I provide valid changes" \
  "I submit an update request" \
  "the stored product reflects and returns the changes"

add_story \
  "Delete a product from the catalog" \
  "catalog administrator" \
  "the ability to delete a product from the catalog" \
  "discontinued products are no longer offered" \
  "Attempting to delete an unknown product returns not found." \
  "a product exists in the catalog" \
  "I submit a request to delete it" \
  "the product is removed and can no longer be retrieved"

add_story \
  "Like a product in the catalog" \
  "authenticated customer" \
  "the ability to like a product" \
  "I can express a positive preference for it" \
  "A customer can record one reaction per product." \
  "a product exists and I have not liked it" \
  "I submit a like request" \
  "my like is recorded and the product's like count increases once"

add_story \
  "Dislike a product in the catalog" \
  "authenticated customer" \
  "the ability to dislike a product" \
  "I can express a negative preference for it" \
  "Like and dislike reactions are mutually exclusive for each customer." \
  "a product exists in the catalog" \
  "I submit a dislike request" \
  "my dislike is recorded and the product's dislike count increases once"

add_story \
  "List all products in the catalog" \
  "customer" \
  "the ability to list all products in the catalog" \
  "I can browse the available selection" \
  "Large result sets are paginated." \
  "the catalog contains products" \
  "I request the product list" \
  "the catalog returns a page of products and pagination information"

add_story \
  "Query a subset of products" \
  "customer" \
  "the ability to query a subset of products" \
  "I can quickly find products matching my needs" \
  "Supported filters include name, category, price, and availability." \
  "the catalog contains products with different attributes" \
  "I query using supported criteria" \
  "only matching products are returned"

add_story \
  "Host the catalog in the cloud" \
  "platform operator" \
  "the catalog service to be hosted in the cloud" \
  "it is available, scalable, and operationally manageable" \
  "Configuration is externalized and the service exposes a health endpoint." \
  "a supported cloud environment is available" \
  "the catalog service is deployed" \
  "the service is reachable and passes its health checks"

add_story \
  "Automate deployment to the cloud" \
  "developer" \
  "new changes to be deployed automatically" \
  "releases are fast, repeatable, and less error-prone" \
  "Deployment runs only after automated tests pass on the default branch." \
  "a change has passed validation and is merged" \
  "the deployment workflow runs" \
  "the new version is deployed and its result is reported"
```
### Build Product Backlog and Icebox
Requirements 7 and 8 go to `Icebox`. Rverything else initially goes to `Product Backlog`.
```bash
for i in "${!ISSUE_URLS[@]}"; do
  gh project item-add "$PROJECT_NUMBER" \
    --owner "$OWNER" \
    --url "${ISSUE_URLS[$i]}" >/dev/null

  if [[ "$i" == "6" || "$i" == "7" ]]; then
    workflow="Icebox"
  else
    workflow="Product Backlog"
  fi

  gh project item-edit "$PROJECT_NUMBER" \
    --owner "$OWNER" \
    --url "${ISSUE_URLS[$i]}" \
    --field "Workflow" \
    --value "$workflow" >/dev/null
done

gh project item-list "$PROJECT_NUMBER" \
  --owner "$OWNER" \
  --limit 100 \
  --field "Workflow" \
  --field "Story points"
```
At this point, the expected distribution is:

| Workflow        | Requirements |
| --------------- | ------------ |
| Product Backlog | 1-6, 9, 10   |
| Icebox          | 7, 8         |
### Configure Kanban view
```bash
gh project view "$PROJECT_NUMBER" --owner "$OWNER" --web
```

In GitHub:
1. Select **New view**.
2. Select **View -> Layout -> Board**.
3. Rename the view *Kanban Board*.
4. Select **View -> Column field -> Workflow**.
5. Ensure all six workflow columns are visible.
6. Save the view by selecting **Save changes**.
### Create Sprint 1 and plan the top four stories
Change `SPRINT_DUE` if necessary.
```bash
SPRINT_TITLE="Sprint 1"
SPRINT_DUE="2026-08-22T15:59:59Z"

MILESTONE_EXISTS="$(
  gh api "repos/$FULL_REPO/milestones?state=all&per_page=100" \
    --jq 'any(.[]; .title == "Sprint 1")'
)"

if [[ "$MILESTONE_EXISTS" != "true" ]]; then
  gh api --method POST "repos/$FULL_REPO/milestones" \
    -f title="$SPRINT_TITLE" \
    -f description="Sprint 1: implement the product catalog CRUD foundation." \
    -f due_on="$SPRINT_DUE" >/dev/null
fi

# Estimates: Create=3, Retrieve=2, Update=3, Delete=2
POINTS=(3 2 3 2)

for i in 0 1 2 3; do
  issue_number="${ISSUE_URLS[$i]##*/}"

  gh issue edit "$issue_number" \
    --repo "$FULL_REPO" \
    --milestone "$SPRINT_TITLE" \
    --add-assignee "@me"

  gh project item-edit "$PROJECT_NUMBER" \
    --owner "$OWNER" \
    --url "${ISSUE_URLS[$i]}" \
    --field "Story points" \
    --number "${POINTS[$i]}" >/dev/null

  gh project item-edit "$PROJECT_NUMBER" \
    --owner "$OWNER" \
    --url "${ISSUE_URLS[$i]}" \
    --field "Workflow" \
    --value "Sprint Backlog" >/dev/null
done
```
The resulting sprint plan is:

| Story            | Points | Workflow       |
| ---------------- | -----: | -------------- |
| Create product   |      3 | Sprint Backlog |
| Retrieve product |      2 | Sprint Backlog |
| Update product   |      3 | Sprint Backlog |
| Delete product   |      2 | Sprint Backlog |

### Optional mock-sprint movement
This leaves story 1 complete, story 2 in progress, and stories 3-4 in the Sprint Backlog.
```bash
# Developers start stories 1 and 2.
for i in 0 1; do
  gh project item-edit "$PROJECT_NUMBER" \
    --owner "$OWNER" \
    --url "${ISSUE_URLS[$i]}" \
    --field "Workflow" \
    --value "In Progress"
done

# Story 1 moves through review and is completed.
gh project item-edit "$PROJECT_NUMBER" \
  --owner "$OWNER" \
  --url "${ISSUE_URLS[0]}" \
  --field "Workflow" \
  --value "Review"

gh project item-edit "$PROJECT_NUMBER" \
  --owner "$OWNER" \
  --url "${ISSUE_URLS[0]}" \
  --field "Workflow" \
  --value "Done"

gh issue close "${ISSUE_URLS[0]##*/}" \
  --repo "$FULL_REPO" \
  --comment "Acceptance criteria satisfied during the mock sprint."
```