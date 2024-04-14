# Conjur integration for Github Action
This repo contains scripts, data files and sample workflow to demonstrate the integration of Github Action and Conjur. 

More explaination of this integration is also explain in below link

https://docs.cyberark.com/conjur-open-source/latest/en/Content/Integrations/github-actions.htm
Comment and discussion, please send to huy.do@cyberark.com

## Prerequisites
You need to have your conjur environment up and running before testing the integration with scripts and workflow here. More detail on how to implement conjur env, please see at https://github.com/huydd79/conjur-poc

Clone this repo to your machine with conjur container running and conjur cli activated and loged in using admin user.

## Configuration
Change the authn-jwt-github.yaml file with the content of your github environment. The rest of file can be kept as default

In this configuration, I am using repository_owner as hostID so all of workflow with same github repository owner can share same id. The host's annotations is using repository_owner_id that can be taking from your github account
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



