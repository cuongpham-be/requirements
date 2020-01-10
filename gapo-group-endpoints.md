# Group Draft APIs

## Group
### Create new group
- Endpoint: `POST /v1/groups`
- Body:
  - name
  - description (optional)
  - category
  - privacy
  - search_privacy

### Update group information
- Endpoint: `POST /v1/groups/:group_id`
- Body:
  - name
  - description
  - location
  - website
  - category
  - privacy
  - search_privacy
  
### Update group cover
- Endpoint: `POST /v1/groups/:group_id/cover`
- Body:
  - file (image)
  
### Block group
- Endpoint: `POST /v1/groups/:group_id/block`

### Archive group
- Endpoint: `POST /v1/groups/:group_id/archive`

## Group members
### Get members:
- Endpoint: `GET /v1/groups/:group_id/members`
- Params:
  - order_by
  - search_pattern
  
### Request to join
- Endpoint: `POST /v1/groups/:group_id/self_request`
- Body:
  - user_id

### Confirm member request
- Endpoint: `POST /v1/groups/:group_id/self_request_confirmation`
- Body:
  - request_id
  - type (accepted, denied)
  
### Invite members
- Endpoint: `POST /v1/groups/:group_id/invitations`
- Body:
  - user_id
  - invited_by (admin, member)
  
### Add member to group
- Endpoint: `POST /v1/groups/:group_id/members`
- Body:
  - user_id

### Remove member from group
- Endpoint: `POST /v1/groups/:group_id/member_removal`
- Body:
  - user_id

### Mute member
- Endpoint: `POST /v1/groups/:group_id/member_mute`
- Body:
  - user_id
  - type (all, notification)

## Group posts

## Group badges

## Group privacy

## Group roles and permissions

## Group search

## Group insights
