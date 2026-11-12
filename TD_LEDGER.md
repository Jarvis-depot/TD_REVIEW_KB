每个 TD case 的证据卡、验收结果、版本和待确认事项

## RIT TD 1.6 — usb_xhci_compliance_RIT_TD_1_6_ss_device

- Review date: 2026-11-12
- Status: **NEED_EVIDENCE**; waiver **not approved**
- User-supplied observation (simulation evidence not independently accessible): after externally waiving a UVM_ERROR, simulation reached completion; original error: `xHCI_Compliance_TD_1.6_PORTPMSC_Register_Test Step- 16.1 READ PORTSC U1 timeout is not RESET successfully. Failed (reg_read_data = 2a1603)`.
- Review source revision (not confirmed to match simulation): USB3_TESTSUITE `f33f727ca4f581a0a15b22524fa453707f62eb08`, `svt/usb_test_suite_svt/X-2025.12/examples/sverilog/tb_dut_usb_3x_host/tests/ts.usb_xhci_compliance_RIT_TD_1_6_ss_device.sv`; RAL definitions `svt/usb_test_suite_svt/X-2025.12/sverilog/src/vcs/svt_usb_test_suite_xhci_ral_model.svp` (U1 [7:0], U2 [15:8], FLA [16], PORTPMSC offset PORTSC+4).
- RTL repository revision inspected only for repository provenance: `Jarvis-depot/usbx_ctl` head `ee2a4f1aaa26271de56d4703828596c973606b3b`; not linked to user's simulation, no confirmed DUT reset implementation.
- Normative source: `xHCI-ComplianceTestSpec-r1_20 CLEAN.pdf`, section 3.1, TD.1.06, steps 10–19. Step 16 is **HCRST**, with Step 16.1 requiring PORTPMSC.U1Timeout=0; Step 16.2 U2Timeout=0; Step 16.3 FLA=0. Step 14 is the separate **hot reset** check.
- Independently checked static fact: `0x002A1603` decoded as PORTSC yields CCS=1, PED=1, PLS=0, PP=1, Port Speed=5, CSC=1, WRC=1, PRC=1. Decoded as PORTPMSC yields U1=0x03, U2=0x16, FLA=0, and nonzero reserved upper bits. The log label `READ PORTSC` raises register-target ambiguity, **not a confirmed root cause**.
- Missing audit evidence: actual simulation TS/RTL commits, plusargs/config, port ID, Step 15 PORTPMSC write/readback, Step 16 USBCMD.HCRST set/clear and USBSTS.CNR, Step 16.1 MMIO read address/data and expected U1[7:0], Step 16.2/16.3, Step 17/18, Gen2 applicability, raw retained UVM_ERROR and final log.
- Conditional review logic: If actual PORTPMSC read after completed HCRST has U1[7:0]=0 but the error used PORTSC or a stale value, consider **VERIFIED_WITH_WAIVER** only after all required steps and other errors are independently validated. If actual PORTPMSC.U1[7:0] remains nonzero, **NEED_FIX**, do not waive. Until observed: **NEED_EVIDENCE**.
- Do not treat this entry as a confirmed root-cause pattern; do not propagate to ISSUE_PATTERNS or COMMON_FLOWS until validated.

