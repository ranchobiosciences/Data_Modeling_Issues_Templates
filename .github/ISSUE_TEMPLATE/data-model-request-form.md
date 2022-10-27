---
name: Data Model Request Form
about: Request changes to the data model
title: "[DM Request]: "
labels: ["Data Model"]
assignees:
  - kwittmeyer
  - jernest1
body:
  - type: markdown
    attributes:
      value: |
        Please describe the changes you would like to see implemented in the data model!
  - type: input
    id: contact
    attributes:
      label: Contact Details
      description: How can we get in touch with you if we need more info?
      placeholder: ex. email@example.com
    validations:
      required: false
  - type: textarea
    id: Description of changes
    attributes:
      label: Description of changes
      description: Please include any relevent examples or data in the text or as an attachment
      placeholder: Tell us what you want to see!
    validations:
      required: true

---
