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

### Leave group
- Endpoint: `POST /v1/groups/:group_id/leave`
- Body:
  - user_id
  
### Modify member role
- Endpoint: `POST /v1/groups/:group_id/member_roles`
- Body:
  - user_id
  - new_role

## Group posts
### Create new post
- Endpoint: `POST /v1/groups/:group_id/posts`

### Edit a post
- Endpoint: `PUT /v1/groups/:group_id/posts/:post_id`

### Remove post
- Endpoint: `DELETE /v1/groups/:group_id/posts/:post_id`

### Pin a post
- Endpoint: `POST /v1/groups/:group_id/pin_post`
- Body:
  - post_id
 
### Toggle post notification
- Endpoint: `POST /v1/groups/:group_id/toggle_post_notification`
- Body:
  - post_id
  - status (on, off)
  
### Toggle post commenting
- Endpoint: `POST /v1/groups/:group_id/toggle_post_commenting`
- Body:
  - post_id
  - status (on, off)

## Group badges
### Assign badges to member
- Endpoint: `POST /v1/groups/:group_id/member_badges`
- Body:
  - user_id
  - badge_ids

### Remove badges from member
- Endpoint: `POST /v1/groups/:group_id/remove_member_badges`
- Body:
  - user_id
  - badge_ids

## Group privacy
### Adjust group privacy
- Endpoint: `POST /v1/groups/:group_id/privacy`

## Group search
### Local search
- Endpoint: `GET /v1/groups/:group_id/search`

## Group insights
### Get group insights
- Endpoint: `GET /v1/groups/:group_id/insights`

