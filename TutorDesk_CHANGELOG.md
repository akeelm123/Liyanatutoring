# TutorDesk 1.1 — changes, verification and limits

Updated 2 November 2026. Open `TutorDesk_v1.1.html` directly in a current browser. It is one standalone file with no external libraries or network dependencies. The original downloaded HTML was left untouched.

## Student context preserved

- Ethan: S$30/hour. Year 3; F1-themed English writing. Twice weekly; times not yet confirmed.
- Mahir: S$30/hour. Grade 6; English writing, grammar and comprehension.
- Nyle: S$30/hour. Poetry and PEEL writing; one lesson per week, time unconfirmed.

No bookings, guardian identities, contact details or payment details were invented. The shipped file starts with these profiles and empty lesson/invoice arrays when no saved records are available. Test records exist only in isolated test-browser storage and intermediate test files.

## Audit findings and implemented fixes

| Area | Finding in v1 | Change in 1.1 |
| --- | --- | --- |
| Recurrence | Only the first occurrence was checked for conflicts | Check all dates before saving; reject the entire series on any overlap. Confirm the occurrence count and date range. |
| Booking logic | Cancelled bookings could trigger overlap warnings; clashes could be overridden | Ignore cancelled lessons; reject active clashes. Adjacent lessons remain allowed. |
| Lesson states | Future lessons could be completed; overnight duration silently wrapped | Completion requires the Singapore end time to have passed. Lessons must finish by midnight. Recurring bookings start as scheduled. |
| Invoice integrity | Invoiced lessons could be edited/deleted | Lock invoiced lessons; prevent duplicate billing; retain billing snapshots. Require explicit sender and payment details for new invoices. |
| Amounts | Rounding differed between completed value and invoices | Calculate lesson charges from rate in cents, rounding each lesson once. Dashboard and invoices use the same calculation. |
| Payments | Unconfirmed one-click state changes; paid PDF still showed total due | Confirm full-payment/reopening actions; record payment timestamp; show total, amount paid and remaining balance separately. Derive overdue labels for unpaid invoices past their due date. |
| PDF | ASCII substitution; long fields overflowed; payment text was limited to six lines | Use browser fonts for Unicode, wrap fields, paginate A4 pages, add invoice/page footers. Show lesson end time, email, paid amount and balance. |
| Calendar | Host timezone contradicted the Singapore label; clickable dates were inaccessible by keyboard; extra lessons were hidden | Anchor today and completion checks to Singapore. Use real date buttons, display every lesson with textual status, label navigation/view controls. Keep wide calendars scrollable within the panel on phones. |
| Student rates | Selecting a free student changed their rate back to S$30 | Preserve a valid zero rate. Reject blank student names. |
| Persistence | No export/restore; corrupt data silently reset; storage errors interrupted saves | Export JSON backups; validate before restore; retain previous browser data before replacement; warn about corrupt/unavailable storage without overwriting it; roll back failed saves. |
| Concurrent editing | A stale tab could overwrite newer records | Observe storage changes and compare the stored snapshot before saving. Warn and require reload when a newer write is detected. |
| Accessibility | Dialog lacked semantics, focus handling and a focus boundary; faint text and tiny targets | Add named dialog, inert background, focus return, Tab containment and Escape close. Add skip link, focus outlines, stronger text contrast, larger controls, labels and live feedback. Avoid accidental backdrop dismissal. |
| Security | No strict import boundary; no network restrictions | Validate record shapes, IDs, references and numeric/date/state fields; escape user text; limit file/text sizes. Restrictive content policy blocks network requests and external scripts/resources. Backups explicitly remain private local files. |

The installed Impeccable and Playwright skills informed the review and testing. The mechanical design scan flagged secondary-button hover contrast, which was fixed. Its font preference and original shadow findings did not warrant replacing the existing visual identity; panel shadows are now removed. This is a targeted accessibility improvement, not a certified WCAG audit.

## Verification performed

Passed interactive Chromium tests against the actual standalone `file://` app in a fresh browser context:

- Original three student profiles and no initial bookings; no uncaught page errors.
- A conflict on the second weekly date rejects the entire new series; four correct dates are saved when clear.
- Twelve weekly dates crossing December into March; adjacent lesson acceptance; midnight crossing rejection; future completion rejection.
- S$30/hour × 75 minutes produces S$37.50; duplicate invoice prevention; invoiced lesson lock.
- Unpaid and paid PDFs download. Paid status and lesson records persist after reload.
- Long Unicode invoice content spans three valid A4 PDF pages. Poppler parsed/rendered the files; first/final unpaid pages and first paid page were visually inspected. Chinese text, final payment text, page numbering, and zero paid balance were present without clipping.
- Student HTML injection is displayed as text; hostile imported IDs are rejected; zero-rate students retain their rate in the booking form.
- Backup download, valid restore, malformed restore rejection, corrupt stored JSON preservation, full-storage rollback and stale-write prevention.
- Existing v1-shaped records saved with JSON indentation remain editable; this is a compatibility check, not a test against any undisclosed personal browser records.
- Singapore date remains correct when the test browser timezone is Los Angeles at a date boundary.
- Desktop screenshot and 390px mobile month/week screenshots; no document-wide horizontal overflow. Calendar content scrolls sideways inside its panel.
- Dialog Tab boundary, Escape dismissal and focus return.
- JavaScript syntax validation.

Safari, Firefox, real touch devices, assistive technology and exhaustive contrast/zoom combinations were not tested. PDF glyph coverage depends on installed fonts.

## Keeping existing browser records when upgrading

The HTML file itself does not contain records entered later: those live in your browser. Version 1.1 uses the original storage key and understands valid v1 records when the browser exposes the same storage.

1. Keep a backup copy of your original HTML and use the same browser/profile.
2. For the best chance of preserving file-based storage, put the updated app at the original file's exact location and filename after making that copy. Keep the original browser tab closed during replacement.
3. Open it and check the students, lessons and invoices before entering anything new. Export a backup immediately if existing records appear.
4. If you see only the three initial profiles, do not assume earlier records were deleted. File-based storage can be isolated by path/browser. Reopen the original app from its previous location/browser and recover its storage first. The old v1 has no export button; recovery may require help exporting its browser storage. A copied HTML file alone cannot migrate those records.
5. Use **Export backup** regularly and **Restore backup** to move validated JSON records to another browser/device. Restore replaces all records after confirmation; it does not merge them.

## Remaining limitations

- Local browser storage and backups are unencrypted and can be lost, cleared or accessed by someone using the same browser profile. No accounts, cloud backup or device sync. Browser storage availability on `file://` varies; heed the warning banner.
- Concurrent-write protection reduces accidental overwrites but does not provide database transactions or a multi-user system. Use one editing tab.
- Recurrence creates separate occurrences. Edit/cancel/delete one at a time; no series-wide update. No overnight lessons, partial payments, credits, refunds, invoice voiding or reissuing. Issued lesson details are locked, so review before issuing.
- PDF pages are raster images: Unicode can render correctly, but text is not searchable/selectable, screen-reader accessible or tagged. Files are larger than text PDFs. They are tutoring invoices, with no tax-registration or tax-compliance claims.
- Changing an invoice to paid updates future downloads. Previously downloaded copies do not update automatically.
- No external calendar sync, booking requests, reminders, payment processing or automatic email sending. Email drafts require manual review and PDF attachment; clipboard failure provides a selectable draft.
- New lesson/invoice validation does not retrospectively repair inconsistent historical v1 records. Invalid backups/storage are rejected and retained for recovery.
