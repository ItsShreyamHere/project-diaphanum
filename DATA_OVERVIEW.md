# Indian Public Procurement Dataset: Release R1 (overview)

Release date: 2026-11-09. Period covered: 2011 to 2026. Scope: nearly every Indian state and union territory (Karnataka is not yet included).

## What it is

The dataset is a record of Indian government procurement. It has two parts:

- **tender notices:** what was put out to tender, by whom, when, and for how much.
- **awards of contract (AOC):** who won and at what value, with the full list of bidders where the portal publishes it.

R1 extends an existing archive compiled by Sarthak Siddhant (about 4.9 million award rows and 4.0 million notice rows), which was
strongest on central-government procurement. R1 adds the state level, where that archive was thin.

| | Sarthak archive | Added in R1 | Combined |
|---|---:|---:|---:|
| Award rows (`aoc_tenders`) | 4,921,960 | 3,850,760 | 8,772,720 |
| Notice rows (`tenders`) | 3,952,191 | 14,987,241 | 18,939,432 |
| Bidder rows of published awards | n/a | 11,064,270 | 11,064,270 |
| Tenders with a bid list but no published award | n/a | 3,446,792 | 3,446,792 |

## Sources

Every record comes from an official government e-procurement portal. Every R1 row carries the URL of the portal page it was read
from, so any record can be checked against its source.

- **NIC GePNIC state portals:** the Government of India's e-procurement system run by the National Informatics Centre.
  - It covers about 30 states and UTs, including West Bengal, Maharashtra, Uttar Pradesh, Kerala, Tamil Nadu, Madhya Pradesh,
    Haryana, Odisha, Punjab, Rajasthan, Delhi, Jammu & Kashmir, Himachal Pradesh, Jharkhand, Uttarakhand, Assam, Tripura, Goa,
    Chandigarh, Puducherry, Ladakh, Andaman & Nicobar, Dadra & Nagar Haveli, Daman & Diu, Lakshadweep, and the North-Eastern
    states.
  - Examples: `wbtenders.gov.in`, `mahatenders.gov.in`, `etender.up.nic.in`, `etenders.kerala.gov.in`, `tntenders.gov.in`.
- **State-run portals** for the states that do not use GePNIC:

| State | Portal |
|---|---|
| Telangana | `tender.telangana.gov.in` |
| Andhra Pradesh | `tender.apeprocurement.gov.in` |
| Chhattisgarh | `eproc.cgstate.gov.in` |
| Bihar | `eproc2.bihar.gov.in` |
| Gujarat | `tender.nprocure.com` |
| Rajasthan | `sppp.rajasthan.gov.in` (work orders, partial) |

- **Karnataka** is delivered separately from the state's own public procurement data, not in this release.

Only publicly accessible portal pages were used. No login-only, paid or leaked material is included.

## Quality controls

- **No duplicates:**
  - each tender appears once in each table;
  - no R1 row repeats a tender already in the Sarthak archive (checked by tender id).
- **No id collisions.** Some state portals use bare numbers as tender ids, and these clash with other portals' ids. Their R1 ids
  carry a state prefix: `TS_` Telangana, `AP_` Andhra Pradesh, `CG_` Chhattisgarh, `GJNP_` Gujarat.
- **Checked against the official dashboard.** R1 notice totals match the state totals on the NIC eProcurement dashboard
  (`eprocure.gov.in/eprocdashboard`) at about 100% for every GePNIC state except Manipur (7%) and Ladakh (97%).
- **Dates validated.** Award dates that did not parse as a real date were left blank rather than guessed.
- **Published and inferred are separate.**
  - Published awards are only those the portal itself published as an award of contract.
  - Where a portal shows the bid list but no award, the bid list is kept in its own table, and the likely winner is in a third
    table, `inferred_awards`.
  - `inferred_awards` is never mixed into the awards. Each of its rows says how the winner was inferred, and carries the note
    "inferred from bid list; award not published".
- **The source archive is untouched.** Sarthak's files were only read. The combined tables mark every row with
  `source` = `sarthak` or `netalink`.

## Files (Parquet, zstd)

### `ours/` (R1 additions only)

| File | What it holds |
|---|---|
| `aoc_tenders`, `aoc_details` | Published awards: tender, organisation, value, award date, winner |
| `award_bidders` | Every bidder on those awards, with status and bid value where published |
| `tenders`, `tender_details`, `tender_details_extra` | Tender notices: title, organisation, value, dates, location, category |
| `bid_lists_no_published_award` | Bid lists of tenders whose award was not published |
| `inferred_awards` | One likely winner per such tender, in 2,928,577 rows. The `method` column gives how it was chosen: |
| | L1 rank: 1,083,948 |
| | sole bidder in the financial round: 1,745,259 |
| | lowest price: 99,370 |

### `combined/` (Sarthak archive + R1)

The same layout as `ours/`. The awards and notices tables hold both sources stacked, in the Sarthak archive's column layout, plus
the `source` column.

## State coverage (awards / notices added in R1)

These are the largest additions:

| State | Awards | Notices |
|---|---:|---:|
| West Bengal | 676k | 2.44M |
| Maharashtra | 665k | 2.19M |
| Uttar Pradesh | 287k | 2.19M |
| Kerala | 484k | 1.42M |
| Rajasthan | 56k | 1.03M |
| Jammu & Kashmir | 4k | 1.07M |
| Tamil Nadu | 138k | 824k |
| Andhra Pradesh | 583k | 20k |
| Telangana | 322k | 25k |

The full state table is in `STATE_TALLY.md`.

## Known limitations

- **Awards are only as complete as what the portals publish.**
  - Several states publish financial-evaluation results without a formal award notice.
  - In Kerala, Tripura, West Bengal, Maharashtra, Goa, Assam, Tamil Nadu, Odisha and Punjab, R1 holds 53–84% of the count shown
    on the NIC dashboard. The remainder is not published per tender.
  - Tamil Nadu does not show bidders before an award.
- **Rajasthan:** state work orders (SPPP) are partial, about 31.5k of about 130k.
- **Manipur:** notices are at about 7% of the dashboard count.
- **Chhattisgarh, Gujarat, Bihar:** awards are few, because these portals publish few award records.
- **Inferred winners are not confirmed awards.** The lowest (L1) bidder usually, but not always, receives the contract, because
  of negotiation, withdrawal or cancellation.
- **Field formats follow each portal.** Values and dates appear as the portal shows them; they are not converted to a single
  currency or format.
