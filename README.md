# Enquiry stage automation for a print job workflow

A working n8n prototype automating the enquiry intake stage of a print job pipeline: from a free-text customer request through to a structured, priced-and-ready order, with AI-based extraction and automatic branching depending on whether the enquiry has enough information to quote.

Built as a proof of concept focused on workflow automation and applied AI in business processes.

## What's in this repo

- `enquiry-workflow.json` : the n8n workflow itself, exportable and importable
- `workflow-canvas.png` : a screenshot of the full workflow canvas
- `print_workflow_template.xlsx` : the Orders database template, including a process map covering the full enquiry-to-production-scheduling journey (see its Read me tab)

## The business problem

Print job enquiries usually arrive as unstructured free text, leaving someone to manually read, interpret, and chase missing details before a quote can even be drafted. This workflow automates the first part of that friction: parsing the request, checking whether it's complete enough to act on, and either logging it as ready for pricing or automatically asking the customer for exactly what's missing.

## How to use the workflow JSON

1. Open n8n (cloud or self-hosted) and create a new workflow.
2. Click the three-dot menu in the top right corner and select **Import from File**, then choose `enquiry-workflow.json`.
3. Reconnect credentials. The export doesn't include secret values, so you'll need to set up your own for:
   - The Chat Model node powering the Information Extractor (this version used Google Gemini's free tier, but any n8n-supported provider works)
   - The Google Sheets nodes (OAuth2, pointed at your own copy of the spreadsheet below)
   - The Send Email node (SMTP credentials, left unconfigured in this version since it's a demonstration build)
4. Make your own copy of `print_workflow_template.xlsx` in Google Sheets, then update the Google Sheets nodes' Document and Sheet fields to point at your copy, since they're currently linked to mine.
5. Test using the sample enquiries below, either by running the form's Test URL directly or executing each node manually with **Execute step**.

This workflow is intentionally left inactive rather than published live, since it's built to demonstrate the logic rather than run unattended in production.

## Workflow walkthrough

**1. On form submission** (trigger)
Captures the customer's name, email, requested deadline, whether they already have artwork ready, and a free-text field asking what they need, written in their own words rather than a long structured form.

**2. Information Extractor**
Uses a connected LLM to parse the free-text field into four structured attributes: `product_type`, `quantity`, `finish`, `sides`. It's instructed to leave a field out entirely rather than guess when the customer didn't mention it, which is what makes the completeness check downstream meaningful rather than just always passing.

**3. Code in JavaScript** (completeness check)
Merges the extracted fields back together with the original form data, generates a unique Order ID, and checks whether all four required fields came through. Outputs an `is_complete` boolean and a `missing_fields` array listing whatever wasn't found.

**4. IF** (branches on `is_complete`)

- **True branch** (enquiry is complete enough to quote): an Append or Update Row node logs the order into the Orders sheet with Status set to "Quote pending," matched on Order ID so later stages can update this same row rather than creating duplicates.
- **False branch** (enquiry is missing details): an Append or Update Row node logs it with Status "Awaiting info" and the specific missing fields recorded, then a Send an Email node sends a clarifying question back to the customer, built dynamically from `missing_fields` rather than a generic template, e.g. "could you let us know the quantity, finish you're after?"

## Sample test data

**Complete enquiry**
> Name: Sarah Mitchell, Email: sarah.mitchell@brightleafmarketing.co.uk, Deadline: 26 June 2026, Have artwork: No
> "Hi, we're launching a new product and need 2000 A5 flyers, double-sided, full colour, gloss finish. We don't have any artwork yet so we'll need help with the design too. Could you let us know pricing and whether you can have these ready by the 26th?"

**Incomplete enquiry**
> Name: James Carter, Email: james.carter@email.com, Deadline: (blank), Have artwork: No
> "Hi, can you do some business cards for us? Need them fairly soon."

## What's next

This covers the Enquiry stage only. The spreadsheet's Read me tab and the accompanying process diagram map out the remaining stages: Estimating (rate-card-based pricing), Order capture, Proofing (document validation, approval tracking, and NLP-parsed change requests), and Production scheduling (capacity-aware slot assignment), each designed to read from and write to the same Orders sheet by Order ID.

## Tools used

n8n, Google Sheets, and an LLM API for structured extraction (in this case, Gemini free api is used).



