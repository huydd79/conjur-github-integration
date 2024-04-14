# Conjur integration for Github Action
This repo contains scripts, data files and sample workflow to demonstrate the integration of Github Action and Conjur. 

More explaination of this integration is also explain in below link

https://docs.cyberark.com/conjur-open-source/latest/en/Content/Integrations/github-actions.htm
Comment and discussion, please send to huy.do@cyberark.com

## Prerequisites
You need to have your conjur environment up and running before testing the integration with scripts and workflow here. More detail on how to implement conjur env, please see at https://github.com/huydd79/conjur-poc

Clone this repo to your machine with conjur container running and conjur cli activated and loged in using admin user.

This repo contains 2 workflow to demonstrate the integration with Conjur using both API key and JWT authentication. For API authentication workflow, you need to get the username and api key from conjur environment and put in github secrets variables. For JWT authentication, the configuration will need to be done as below.

## Configuration for JWT authentication
Conjur JWT authentication from github action will be done using container image conjur-action (https://github.com/cyberark/conjur-action). The JWT value will be retrieaved using curl command as below:

```JWT_TOKEN=$(curl -H "Authorization:bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" "$ACTIONS_ID_TOKEN_REQUEST_URL" | jq -r .value )```

Below are sample of github JWT content, you can base on this content to build up your customization for JWT authentication restriction
```
{
  "jti": "99c2cf07-905b-41b7-a179-08c5fd17abba",
  "sub": "repo:huydd79/test:ref:refs/heads/main",
  "aud": "https://github.com/huydd79",
  "ref": "refs/heads/main",
  "sha": "37e804e2c3d53a01264ac9dedd5f9a59eacbd0a3",
  "repository": "huydd79/test",
  "repository_owner": "huydd79",
  "repository_owner_id": "86530496",
  "run_id": "8678864795",
  "run_number": "19",
  "run_attempt": "2",
  "repository_visibility": "public",
  "repository_id": "380556427",
  "actor_id": "86530496",
  "actor": "huydd79",
  "workflow": "Conjur-Authn-JWT",
  "head_ref": "",
  "base_ref": "",
  "event_name": "workflow_dispatch",
  "ref_protected": "false",
  "ref_type": "branch",
  "workflow_ref": "huydd79/test/.github/workflows/conjur-jwt-test.yml@refs/heads/main",
  "workflow_sha": "37e804e2c3d53a01264ac9dedd5f9a59eacbd0a3",
  "job_workflow_ref": "huydd79/test/.github/workflows/conjur-jwt-test.yml@refs/heads/main",
  "job_workflow_sha": "37e804e2c3d53a01264ac9dedd5f9a59eacbd0a3",
  "runner_environment": "github-hosted",
  "iss": "https://token.actions.githubusercontent.com",
  "nbf": 1713080246,
  "exp": 1713081146,
  "iat": 1713080846
}
```
With above understanding of github jwt attribute, you need to change the authn-jwt-github.yaml file with the content of your github environment. The rest of file can be kept as default

In this configuration, I am using repository_owner as hostID so all of workflow with same github repository owner can share same id. The host's annotations is using repository_owner_id that can be taking from your github account.
```
- !policy
  id: jwt-apps/github
  owner: !group github-admins
  body:
  - !layer
  - &github-hosts
    - !host
    - !host
      id: huydd79
      annotations:
        authn-jwt/github/repository_owner_id: 86530496
  - !grant
    role: !layer
    members: *github-hosts
```
With that, run 01 and 02 scripts to load conjur policy and enable jwt authentication for github.

Script 03 will put github jwt authenticaiton parameters to conjur variable. 

```
# This is public URL for jwt public key - jwks validation
conjur variable set -i conjur/authn-jwt/github/jwks-uri -v https://token.actions.githubusercontent.com/.well-known/jwks
# This is setting to choose which jwt attribute will be used as hostID/userID for authentication
conjur variable set -i conjur/authn-jwt/github/token-app-property -v repository_owner
# This is setting to set the hostID/userID path in conjur env: for example with below setting, the host in conjur will be started with jwt-apps/github
conjur variable set -i conjur/authn-jwt/github/identity-path -v jwt-apps/github
# This is setting for issue URL
conjur variable set -i conjur/authn-jwt/github/issuer -v https://token.actions.githubusercontent.com
# This is setting for the mandatory attribute that need to have in jwt content. This attribute value also need to be matched to host's annotations in policy.
conjur variable set -i conjur/authn-jwt/github/enforced-claims -v "repository_owner_id"
```
## Testing with Github Action
Creating github action workflow to test for conjur communication. You can copy two yaml file from this repo to your environment and run it.

## Huy.Do




