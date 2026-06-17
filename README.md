# Enquiry stage workflow automation

A working n8n prototype automating the enquiry intake stage of a print 
job pipeline: from a free-text customer request through to a structured, 
priced order ready for production.

## What's in this repo
- `enquiry-workflow.json` — the n8n workflow (form trigger → AI extraction 
  → completeness check → branch into quote-ready or clarification-needed)
- `workflow-illustration.png` — screenshot of the full workflow
- `print_workflow_template.xlsx` — the Orders database template, with a 
  process map covering the full enquiry-to-production-scheduling journey
- `n8n enquiry stage.json` — json file of the full workflow
  

## What this workflow demonstrates
- NLP-based document understanding (n8n's Information Extractor node 
  pulling structured fields from free text)
- Conditional automation logic (branching on data completeness)
- End-to-end process design across enquiry, estimating, order capture, 
  proofing, and production scheduling (see the `print_workflow_template.xlsx`'s Read me tab)
