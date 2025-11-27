# API Endpoints Summary

## All Endpoints

| Title | Namespace | Endpoint | Description |
|-------|-----------|----------|-------------|
| **Mailbox Management** |
| List Mailbox Integrations | Mailbox | `GET /integrations/mailboxes` | Get all mailbox integrations for the current user with their settings and status |
| OAuth Authorization | Mailbox | `GET /auth/google` | Exchange OAuth authorization code for access tokens and create/update integration |
| Update Integration Settings | Mailbox | `PATCH /integrations/:id` | Update rate limits or status for a mailbox integration |
| **Sequences** |
| List Sequences | Sequences | `GET /sequences` | List all sequences for the authenticated user with statistics |
| Create Sequence | Sequences | `POST /sequences` | Create a new sequence with instructions and email templates |
| Get Sequence | Sequences | `GET /sequences/:id` | Load a specific sequence with all configuration details |
| Update Sequence | Sequences | `PUT /sequences/:id` | Update all fields of a sequence (full object replacement) |
| Update Sequence Status | Sequences | `PATCH /sequences/:id` | Update only the status of a sequence (activate/pause/archive) |
| List Sequence Leads | Sequences | `GET /sequences/:sequenceId/leads` | List all leads enrolled in a sequence with their status and statistics |
| Enroll Leads in Sequence | Sequences | `POST /sequences/:sequenceId/leads` | Bulk enroll leads into a sequence with initial status set to active |
| Update Sequence Lead | Sequences | `PATCH /sequences/:sequenceId/leads/:leadId` | Update a lead's introducers, status, or next follow-up time |
| Remove Lead from Sequence | Sequences | `DELETE /sequences/:sequenceId/leads/:leadId` | Remove a lead from a sequence by setting status to finished |
| Get Lead Activities | Sequences | `GET /sequences/:sequenceId/leads/:leadId/activities` | Get all activity history for a specific lead in a sequence |
| List Sequence Emails | Sequences | `GET /sequences/:sequenceId/emails` | List all outgoing emails (draft, scheduled, and sent) for a sequence |
| Get Lead Email History | Sequences | `GET /sequences/:sequenceId/leads/:leadId/emails` | Get complete email history (outgoing + incoming) for a specific lead, organized by threads |
| Update Scheduled Email | Sequences | `PATCH /sequences/:sequenceId/emails/:emailId` | Edit a draft or scheduled email's content or send time (only if not yet sent) |
| Cancel Scheduled Email | Sequences | `DELETE /sequences/:sequenceId/emails/:emailId` | Cancel a scheduled email by setting status to failed with cancellation metadata |
| List Pending Activities | Sequences | `GET /sequences/:sequenceId/activities` | List pending activities awaiting user approval (typically outgoing email drafts) |
| Update Activity | Sequences | `PATCH /sequences/:sequenceId/activities/:activityId` | Update activity content, scheduled time, or approval status (approve/reject) |

## Notes

- All leads, emails, and activities are under the **Sequences** namespace
- Mailbox endpoints remain at `/integrations` and `/auth/google`
- Email Templates endpoint has been removed
- Webhook endpoints have been removed
- Approve and Reject have been consolidated into the Update Activity endpoint using the `status` field
- Path parameters follow the pattern: `:sequenceId`, `:leadId`, `:emailId`, `:activityId`

## Total Endpoints: 19

- **Mailbox:** 3 endpoints
- **Sequences:** 16 endpoints (sequences + leads + emails + activities)
