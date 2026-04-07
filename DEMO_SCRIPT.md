# BookLeaf Cover Validation — Demo Script
**Estimated recording time: 5–7 minutes**

---

## BEFORE YOU START — SETUP CHECKLIST

- [ ] n8n open with the workflow visible (zoomed out to show all nodes)
- [ ] Airtable open in a browser tab (Authors table visible)
- [ ] Google Drive open — Incoming_Covers folder ready
- [ ] Gmail open — inbox visible
- [ ] The test cover image ready to upload (`9789898652364_text.png`)
- [ ] Screen recorder running, mic levels checked

---

## SECTION 1 — INTRO (30 seconds)

> "Hey, I'm going to walk you through BookLeaf's automated cover validation system.
> 
> The problem it solves: every time an author submits a book cover, someone has to manually check whether the layout follows our specs — specifically whether the author name is staying out of the award badge zone at the bottom. That takes time, and it's easy to miss.
> 
> This system does that automatically — the moment a cover lands in Google Drive, it gets analyzed by AI, logged in Airtable, and the author gets an email. No manual review needed."

---

## SECTION 2 — AIRTABLE TABLES (1 minute)

*Switch to Airtable tab*

> "There are two tables powering this."

**Show the Authors table:**

> "First — the Authors table. This is where all our author records live. Each row has the author's name, email, ISBN, and book title. When a cover comes in, the workflow looks up the author by ISBN so it knows who to email."

*Point to a row — highlight Author_Name, Author_Email, ISBN columns*

> "The ISBN is the link between the cover file and the author — the filename itself must follow our naming convention: 13-digit ISBN, underscore, the word 'text', dot PNG or PDF. That's how the system knows who submitted."

**Switch to Book_Cover_Validations table:**

> "Second table — this is the validation log. Every cover that gets processed creates a record here automatically. You can see the status — PASS or REVIEW NEEDED — the confidence score the AI gave, any issues it found, and the correction instructions that go straight into the email."

*Scroll across to show Status, Confidence_Score, Issues_JSON, Correction_Instructions columns*

> "So your team always has a full audit trail. You can filter by status, by ISBN, by date — whatever you need."

---

## SECTION 3 — THE WORKFLOW (2 minutes)

*Switch to n8n tab — workflow visible*

> "Now let's walk through the workflow. There are 15 nodes and the whole thing runs automatically — no one clicks anything."

*Start from the left, point to each node as you mention it*

**Watch Incoming Covers:**
> "It starts here — a Google Drive trigger that polls the Incoming Covers folder every minute. The moment a new file appears, everything fires."

**Extract ISBN:**
> "This node reads the filename and extracts the ISBN using a regex. If the filename doesn't follow the convention — wrong format, missing ISBN — it stops right here and throws an error. No bad data gets through."

**Check File Type → PDF branch vs PNG branch:**
> "Next it checks the file type. PNG goes straight through. PDF gets converted to PNG first using CloudConvert — we need an image to send to the AI, not a document. Both paths merge back together here."

**Convert to Base64:**
> "The image gets converted to a base64 data URI — that's the format the OpenAI Vision API needs to actually see the image."

**AI Cover Validator:**
> "This is the core node. It sends the image directly to GPT-4o with a very specific prompt. The AI is told to focus on the front cover only, look at the bottom badge zone, check if the author name is intruding into it, check margins, check image quality — and return structured JSON. Not a paragraph of text — actual machine-readable JSON so the workflow can act on it."

**Parse AI Results:**
> "This node reads the AI's JSON response and applies a final rule: to get a PASS, the confidence must be 90% or higher AND there must be no critical issues. If either fails, it becomes REVIEW NEEDED. This stops borderline cases from slipping through."

**Lookup Author → Merge Author Data:**
> "It looks up the author in Airtable by ISBN, then merges that data — name, email, book title — with the validation result."

**Create Airtable Record:**
> "One record gets written to the validation log — status, score, issues, file URL. Permanent audit trail."

**Route by Status → Send Email:**
> "Finally, it routes to one of two email nodes. PASS goes one way, REVIEW NEEDED goes the other. The author gets the right email automatically."

---

## SECTION 4 — LIVE DEMO (2 minutes)

*Switch to Google Drive — Incoming_Covers folder*

> "Let me show you this live. I'm going to upload this cover right now."

*Upload the test file — `9789898652364_text.png`*

> "File is in. The workflow polls every minute, so within about 60 seconds we should see it fire."

*Switch to n8n — watch for execution to appear in the execution list*

> "There it goes — you can see the execution starting. Each node lights up green as it completes."

*Watch the execution run through — point out nodes activating*

> "Extract ISBN picked up the filename. Image downloaded. File type check — it's a PNG so we skip the PDF conversion. Base64 conversion. Now it's hitting the AI..."

*Wait for completion*

> "Done. Let's check the results."

*Switch to Airtable — Book_Cover_Validations table — refresh*

> "New record — right here. ISBN matches. Status: PASS. Confidence score: 93%. Badge area clear: true. No issues."

*Switch to Gmail*

> "And the author got their email. Subject line: 'Your Book Cover Has Been Approved'. Name, book title, ISBN, confidence score — all pulled in dynamically."

*Open the email to show the full content*

> "If it had failed, they'd get a revision email listing exactly what's wrong and how to fix it — with their ISBN in the resubmission filename already filled in."

---

## SECTION 5 — CLOSE (30 seconds)

> "That's the full system. Cover uploaded, AI analyzed it, Airtable logged it, author emailed — in under 60 seconds, completely automated.
> 
> What used to take manual review time now just runs. And every decision is logged so there's a full record if an author ever questions the result.
> 
> The only cost is the OpenAI API call — about 6 cents per cover for GPT-4o. For a publishing operation processing dozens of covers a month, that's a few dollars total."

---

## RECORDING TIPS

- **Zoom in** on Airtable column headers when pointing to them — small text is hard to see in recordings
- **Pause 2 seconds** after each node in the workflow before moving to the next one
- **Slow your mouse** when hovering over nodes — viewers need time to read the node names
- **Refresh Airtable** dramatically (make it visible) to show the record appearing in real time
- If the workflow takes more than 90 seconds, cut and paste the clip — don't make viewers watch a loading spinner
