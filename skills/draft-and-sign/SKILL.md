---
name: draft-and-sign
description: Draft a new contract in GitLaw from a template or a description, review it with the user, and send it for e-signature. Use when the user asks to create, draft, or prepare an agreement and get it signed.
---

# Draft a contract and send it for signature

1. If you have not called `guide` in this session, call it first.
2. **Gather the facts.** Ask for what you need: document type, parties, jurisdiction, and key
   terms (dates, amounts, duration). `list_contacts` finds parties the user already has.
3. **Look for a template.** Call `search_templates` with the document type (and `jurisdiction`
   if known). If a result fits, tell the user which one you will use.
4. **Draft.** Call `draft_document` with `instructions` describing the parties and terms, the
   `templateId` if you chose one, and `jurisdiction` if known. Reuse the `chatId` and
   `organizationId` from earlier AI calls. Drafting can take minutes. Show the returned `url`.
5. **Review with the user.** Read the draft with `get_document` and summarise it. For questions
   about whether terms are standard or risky, call `ask` with the `fileId` and present GitLaw's
   answer. Apply requested changes with `edit_document`.
6. **Confirm signers.** Show the user each signer's name and email, listed in the order the
   parties appear in the document, and get an explicit yes. Signing emails go to people outside
   GitLaw.
7. **Send.** Call `send_for_signature` with the `fileId` and the confirmed `signers`. If it
   returns `VALIDATION_ERROR` because signature blocks are missing, tell the user what the error
   says and give them the returned `url`. Use `fieldsAtBottom: true` only if the user asks.
8. **Track.** Keep the returned `requestId`. Use `get_signing_status` when the user asks about
   progress, and `export_signed_pdf` once the request is completed.
