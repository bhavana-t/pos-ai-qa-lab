# v1 Test Case Comparison — BRD-POS-017 (Refund to Original Tender)

| Run | # cases | Rules covered (R1–R8) | Boundary cases (29/30/31 days, $499.99/$500/$500.01) | Invented details | Valid JSON |
|---|---|---|---|---|---|
| **1** | 37 | 8/8 (R1:4, R2:5, R3:4, R4:5, R5:5, R6:4, R7:5, R8:5) | Days: ❌29, ✅30, ✅31 (also day 0, 60)<br>Amount: ❌$499.99, ✅$500, ✅$500.01 | Medium: cash drawer opens, manager ID recorded, "insufficient permissions" message, tenderType `VISA`/`GIFT_CARD`, storeId `S001`, "$50.00 shown as max remaining", error log includes originalTxnId + timestamp, no full card number in message | ✅ Yes |
| **2** | 33 | 8/8 (R1:3, R2:4, R3:4, R4:5, R5:5, R6:4, R7:4, R8:4) | Days: ❌29, ✅30, ✅31 (also day 0)<br>Amount: ❌$499.99, ✅$500, ✅$500.01 | Medium–High: masked PAN, cash drawer opens, "refund window expired" text, receipt shows refund legs and store-credit ID/balance, manager ID recorded, tenderType `CARD`, storeId `1234`, SLA-breach alert, reprocess job. **Flags 3 assumptions with "confirm with BA"** | ✅ Yes |
| **3** | 40 | 8/8 (R1:4, R2:5, R3:5, R4:7, R5:6, R6:4, R7:5, R8:4) | Days: ❌29, ✅30, ✅31 (also day 0, 90)<br>Amount: ✅$499.99, ✅$500, ✅$500.01 | High: TXN IDs (`TXN-10001`…), Visa ending `4417`, storeId `0421`, receipt shows `CASH`/`STORE_CREDIT`, "refund window expired" and "transaction not found" texts, manager ID recorded, tenderType `VISA`, HTTP 500 | ✅ Yes |

## Notes

- **Day 29 is missing in all three runs.** Every run tests day 0, 30, and 31 instead. Day 29 is low value, since day 30 being allowed already implies day 29 is.
- **Only Run3 has all three amount boundaries.** Runs 1 and 2 skip $499.99.
- **Invented details:** Run3's concrete values (TXN IDs, card ending, store number) make tests easier to execute but aren't from the BRD. Run2 is the only run that labels its assumptions as needing BA confirmation. All three runs invent a manager-ID requirement and specific error-message text that the BRD never defines. The `tenderType` value also disagrees between runs (`CARD` in Run2, `VISA` in Runs 1 and 3).
- **JSON check:** All three files parse as valid JSON arrays.
