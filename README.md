# get-team-members

This action fetches the users of a GitHub team.

## Usage

```yaml
- uses: garnertb/get-team-members@v1
  with:
    # Name of the organization that the team is a part of.
    # (required)
    org: ''

    # Name of the team to get the members of.
    # (required)
    team_slug: ''

    # The role to filter team members by.
    # Must be one of: 'member', 'maintainer', 'all'.
    # Default: 'all'
    role: ''

    # Personal access token (PAT) used to fetch team members. 
    #
    # Default: ${{ github.token }}
    token: ''
```

## Scenerios

### Fetch handles for all members of a team

```yaml
- id: get-members
  uses: garnertb/get-team-members@v1
  with:
    org: garnertb-io
    team_slug: ops
    token: ${{ secrets.GITHUB_TOKEN }}

- run: "echo ${{ steps.get-members.outputs.members }}"
  shell: bash
```

### Fetch handles for the maintainers of a team

```yaml
- id: get-members
  uses: garnertb/get-team-members@v1
  with:
    org: garnertb-io
    team_slug: ops
    role: maintainer
    token: ${{ secrets.GITHUB_TOKEN }}

# print results
- run: "echo ${{ steps.get-members.outputs.members }}"
  shell: bash
```

### Access additional member properties

[Raw member data](https://docs.github.com/en/rest/teams/members#list-team-members) is accessible via the `data` output and can be manipulated with `jq`.

```yaml
- id: get-members
  uses: garnertb/get-team-members@v1
  with:
    org: garnertb-io
    team_slug: ops
    role: maintainer
    token: ${{ secrets.GITHUB_TOKEN }}

  - run: "echo '${{ steps.get-members.outputs.data }}' | jq '.[] | {login: .login, id: .id}'"
    shell: bash
```

Results:

```json
{
  "login": "garnertb",   
  "id": 1234
}
```

### Assign team members to issue

Use this action to set assignees for a new issue with [issue-bot](https://github.com/imjohnbo/issue-bot).

```yaml
- id: get-members
  uses: garnertb/get-team-members@v1
  with:
    org: garnertb-io
    team_slug: ops
    role: maintainer
    token: ${{ secrets.GITHUB_TOKEN }}

- name: Create new issue
  uses: imjohnbo/issue-bot@v3
  with:
    assignees: ${{ steps.get-members.outputs.members }}
    title: Hello, world
    body: |-
      :wave: Hi, {{#each assignees}}@{{this}}{{#unless @last}}, {{/unless}}{{/each}}!
```
