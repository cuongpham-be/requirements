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

## Group members

## Group posts

## Group invitations

## Group badges

## Group privacy

## Group roles and permissions

## Group search

## Group insights
