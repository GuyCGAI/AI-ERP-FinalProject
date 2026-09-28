# 6. Tax documents and Israeli VAT

The system applies basic Israeli tax rules (section 10 of the course brief). Where each rule lives:

| Rule | Where it's enforced |
|---|---|
| **VAT 18%** from 01/01/2025, **17%** before | WF13 `חישוב מע״מ` (documents created in the app) · WF9's agent prompt (documents created by the manager bot) · WF1 `Read Document` + `Validate` (every document, whoever created it) |
| **Running document numbers** — every document gets the next number | WF13 `מספר רץ הבא` (highest `DocNumber` + 1, starting at 1001) · WF9 searches the last number before creating |
| **Document types are kept apart** — invoice (חשבונית עסקה), tax invoice (חשבונית מס), receipt (קבלה) | three tables (Invoices, TaxInvoices, Receipts), one Airtable trigger each in WF1, and a title per type in WF8 |
| **A tax invoice needs the customer's business number** | WF1 `Validate` |
| **Amounts add up** | WF1 `Validate`: `VATAmount = Subtotal × rate`, `Total = Subtotal + VATAmount` (to the agora) |

## How a document is checked

WF1's `Validate` node collects every problem into one text; an empty text means the document is valid. Each check and the message it writes into `ValidationError`:

| Check | Applies to | Message |
|---|---|---|
| `DocNumber` > 0 | all | `DocNumber חסר או לא חיובי` |
| a customer (`CustomerId` or `CustomerName`) | all | `חסר לקוח (CustomerId או CustomerName)` |
| `IssueDate` set | all | `IssueDate חסר` |
| `Amount` > 0 | receipts | `Amount חייב להיות חיובי` |
| `Subtotal` > 0 | invoices, tax invoices | `Subtotal חייב להיות חיובי` |
| `VATRate` = the rate for the issue date | invoices, tax invoices | `VATRate צריך להיות 0.18` |
| `VATAmount` = Subtotal × rate | invoices, tax invoices | `VATAmount לא תואם Subtotal*VATRate` |
| `Total` = Subtotal + VATAmount | invoices, tax invoices | `Total לא תואם Subtotal+VATAmount` |
| `CustomerBusinessNumber` set | tax invoices | `בחשבונית מס חובה מספר עוסק של הלקוח` |

A valid document is queued in `Files` and rendered by WF8; an invalid one gets `Status = Invalid` and the message, so the owner sees exactly what to fix.

## Worked example — INV-1003

Created from the app with a customer and an amount only:

| Field | Value | Written by |
|---|---|---|
| `InvoiceId` / `DocNumber` | INV-1003 / 1003 | WF13 — running number |
| `Subtotal` | 3,200.00 ₪ | the form |
| `VATRate` | 18% | WF13 — issue date 29/09/2026 ≥ 01/01/2025 |
| `VATAmount` | 576.00 ₪ | WF13 — 3,200 × 0.18 |
| `Total` | 3,776.00 ₪ | WF13 |
| validation | passed | WF1 |
| `PdfUrl` | Drive link to `Invoice-1003.html` | WF8 |

## The document

The template is Hebrew, right-to-left HTML inside WF8's `Build HTML` node (readable copies: [`templates/invoice.html`](../templates/invoice.html), [`tax-invoice.html`](../templates/tax-invoice.html), [`receipt.html`](../templates/receipt.html)). WF8 fills it, turns it into a `text/html` file, uploads it to Google Drive, and writes the link to `PdfUrl`; the app only shows the link.

![INV-1003](screenshots/invoice-1003.jpg)

**And the PDF?** The project uses no external conversion service, so there is no server to set up. For a PDF: in Drive, right-click the file → Open with → Google Docs → File → Download → PDF.

## Known limitation

Running numbers can collide if two documents are created within the same polling window — the "highest number + 1" is read before either is written. A full fix would need a counter table with locking.
