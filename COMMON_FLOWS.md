已确认的 xHCI、Transfer、Sequence、RTL 公共调用链

## X-2025.12 — Distinguish xHCI Transfer Ring vs Command Ring stimulus (static source verified, 2026-11-13)

- Verified GitHub source version: `Jarvis-depot/USB3_TESTSUITE@f33f727ca4f581a0a15b22524fa453707f62eb08`.
- Transfer path: `svt_usb_transfer` sent to `p_sequencer.host_virt_sequencer.xfer_sequencer.execute_item(xfer)` reaches xHCI driver transfer processing / `prepare_transfer_trbs`, where a CONTROL_TRANSFER with predefined trb[0] must have `trb_type=SETUP_STAGE` (`svt_usb_test_suite_xhci_driver.svp` lines 7522–7533). **Not suitable for testing a Command Ring LINK TRB**.
- Command-service path: `svt_usb_agent_service::XHCI_EXECUTE_COMMAND` processed by driver `do_agent_service()` lines 13101–13185 → `issue_command()` lines 9967–10271 writes a supported Command TRB into Command Ring memory, rings `Doorbell_0`, then `wait_for_command_completion_event()`. But `issue_command` does NOT handle explicit LINK TRB as a supported command name and the service unconditionally waits; do not call it as a complete TD2.11 fix.
- Existing lower-level sequence helpers for purpose-built ring tests: `svt_usb_xhci_system_virtual_sequence_collection.svp` lines 959–1016 `system_memory_access("SYS_MEM_WRITE", byte_count, address, byte_array)` sends SYSTEM_MEM_WRITE service; lines 798–847 `xhci_register_access("REG_WRITE","Doorbell_Register","Doorbell_0", write_data, read_data)` sends named register write. Ring-address allocation, producer cycle, command-state tracking and event checks must be maintained by the caller; helper existence alone does NOT establish successful TD2.11 execution.
- Distinguish DUT command-doorbell and DMA fetch from generic USB LTSSM/U0: xHCI TRB ring tests may require no connected USB device. Runtime wave/log verification pending.

## X-2025.12 — Host/Device transfer scoreboard end-of-run queue diagnosis

- Static source verified at USB3_TESTSUITE@f33f727ca4f581a0a15b22524fa453707f62eb08, `svt/usb_test_suite_svt/X-2025.12/examples/sverilog/tb_dut_usb_host/env/usb_transfer_scoreboard.sv`. Verify compiled environment uses this variant before applying conclusions.
- `write_host_xfer_ended()` lines 99–145 and `write_dev_xfer_ended()` lines 76–95 push completed transfers into separate queues. `post_xfer_sb_compare()` lines 350–407 takes Device ENDED queue as driver, matches Host ENDED or ABORTED by xfer type, device address, endpoint, stream (and control direction), then compares/removes matched transfers.
- `report_phase()` lines 497–507 reports match/mismatch counters; if either ENDED queue remains, `empty_queue_chk.register_fail` prints `Scoreboard is not Empty` and lists `Host xfers[i]` / `Device xfers[i]`. Aborted-only residuals are reported as notes (lines 510+), not the same ERROR.
- Diagnostics: isolate remaining Host xfer identity/EP/length/status and determine whether Device VIP emitted matching ENDED or ABORTED, then correlate with target TRB/doorbell/endpoint event. An error reported in `report_phase` is necessarily late in UVM phases but gives no evidence that the original unmatched transfer condition arose late. This code path is reused across different TD cases but is **not itself a verified root-cause attribution or waiver**.

