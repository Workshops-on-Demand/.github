v1.0.2
- Adds HPE Developer Community Meetup 2025 (wod)
- private-public uses the new wod-install (wod-private)
- templates of content for a private repo (wod-private)
- Fix #7 when remaining apt process exists and install fails (wod-install)
- Remove the dotnet and Ijava kernel and tool setup to do it in private (wod-backend)
- Fix #4 by using JPHUB var to amend .profile (wod-backend)
- Fix #9 as ansible variables are not expanded after the raw instruction (wod-api-db)
- Force pg17 as 18 create issues with no_null for now (wod-api-db)

v1.0.1
- Manage workshop badges either by URL or local file hosted on the frontend.
- Remove now unavailable references to pam_lastlog if present (wod-install)
- Adds script to build automatically VMs with virt-lightning (wod-install)
- Declare a new WODDISTRIBNAME variable (wod-install)
- Use the new WODDISTRIBNAME variable to avoid fixed distribution name in apt repo (wod-backend)
- Manage badges names locally on the frontend or with a full URL (wod-api-db)
- Fix jupurl computation to correct e-mails sent (wod-api-db)
- Adds badge images (wod-frontend)
- FLOSS WKSHP Badges are now hosted on the frontend (wod-notebooks)

v1.0.0
- First full features version
