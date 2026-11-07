# Building a Defender Security Dashboard

A Microsoft Defender portal workbook that shows device sensor health, Tier 0 identity server coverage, protection control compliance, antivirus mode, alerts and attack surface reduction (ASR) readiness on one page.

Built entirely on **advanced hunting** - no Azure resources, data export or scripts. Every section queries live data each time the workbook opens or refreshes.

Companion repository for the **Building a Defender Security Dashboard with Workbooks** series on [cloud-secure.ai](https://cloud-secure.ai/):

- Introduction - REPLACE_WITH_INTRO_URL
- Part 1 - Workbooks in the Defender portal, and device health - REPLACE_WITH_PART1_URL
- Part 2 - Are the protections actually on? - REPLACE_WITH_PART2_URL
- Part 3 - Alerts and attack surface reduction - REPLACE_WITH_PART3_URL

## Repository layout

```
workbook/
  DefenderDashboard.workbook        The complete workbook (Advanced editor JSON)
queries/
  part1-device-health/              Sensor health and Tier 0 coverage
  part2-protection-controls/        Control compliance, antivirus mode, EDR gaps
  part3-alerts-asr/                 Alerts and attack surface reduction
```

## Query reference

Every `.kql` file runs as-is in **Microsoft Defender advanced hunting**. Each file starts with a comment block describing what it answers and which tables it uses.

| Query | Section | Question it answers | Tables |
|---|---|---|---|
| [1-1-sensor-summary.kql](queries/part1-device-health/1-1-sensor-summary.kql) | Sensor health at a glance | How many onboarded devices are active, not seen for 7+ days, or unhealthy? | DeviceInfo |
| [1-2-sensor-by-class.kql](queries/part1-device-health/1-2-sensor-by-class.kql) | Sensor health by device class | Which device classes have sensors that are not reporting or unhealthy? | DeviceInfo |
| [1-3-inactive-devices.kql](queries/part1-device-health/1-3-inactive-devices.kql) | Devices not reporting a healthy sensor | Which devices have stopped reporting, and for how long? | DeviceInfo |
| [1-4-identity-servers.kql](queries/part1-device-health/1-4-identity-servers.kql) | Tier 0 identity servers | Are our domain controllers, AD CS, AD FS and Entra Connect servers onboarded and healthy? | DeviceInfo |
| [1-5-onboarding-gap.kql](queries/part1-device-health/1-5-onboarding-gap.kql) | Devices that can be onboarded | How many discovered devices are not yet onboarded, by platform? | DeviceInfo |
| [2-1-control-matrix.kql](queries/part2-protection-controls/2-1-control-matrix.kql) | Control compliance % by device class | Which protection controls are on, for each class of device? | DeviceInfo, DeviceTvmSecureConfigurationAssessment, DeviceTvmSecureConfigurationAssessmentKB |
| [2-2-weakest-controls.kql](queries/part2-protection-controls/2-2-weakest-controls.kql) | Controls with the most non-compliant devices | Where will one policy change fix the most devices? | DeviceTvmSecureConfigurationAssessment, DeviceTvmSecureConfigurationAssessmentKB |
| [2-3-av-mode.kql](queries/part2-protection-controls/2-3-av-mode.kql) | Antivirus mode by platform | Which devices run Defender Antivirus in passive or EDR block mode? | DeviceTvmInfoGathering, DeviceInfo |
| [2-4-tamper-av-gaps.kql](queries/part2-protection-controls/2-4-tamper-av-gaps.kql) | Devices failing antivirus, tamper protection or EDR controls | Which devices fail the controls an attacker turns off first? | DeviceTvmSecureConfigurationAssessment, DeviceTvmSecureConfigurationAssessmentKB |
| [3-1-alerts-by-source.kql](queries/part3-alerts-asr/3-1-alerts-by-source.kql) | Alerts by source and severity | Which products are generating alerts, and how severe are they? | AlertInfo |
| [3-2-alerts-trend.kql](queries/part3-alerts-asr/3-2-alerts-trend.kql) | Daily alerts by source | How has alert volume changed day by day over 30 days? | AlertInfo |
| [3-3-top-alerts.kql](queries/part3-alerts-asr/3-3-top-alerts.kql) | Top 20 alert titles | Which alerts keep coming back? | AlertInfo |
| [3-4-asr-by-rule.kql](queries/part3-alerts-asr/3-4-asr-by-rule.kql) | ASR events by rule and mode | Which ASR rules are blocking, and which are only auditing? | DeviceEvents |
| [3-5-asr-ready-to-block.kql](queries/part3-alerts-asr/3-5-asr-ready-to-block.kql) | Audit-mode ASR rules by devices affected | Which audit-mode ASR rules affect the fewest devices and are easiest to move to block? | DeviceEvents |

## Option 1 - Import the workbook

1. In the Defender portal, go to **Workbooks** > **+ Add workbook** > **Edit**.
2. Select **Advanced editor**, replace the JSON with the contents of `workbook/DefenderDashboard.workbook` and select **Apply**.
3. Set the **Tier 0 servers** parameter to the short host names of your domain controllers, AD CS, AD FS and Entra Connect servers, separated by commas.
4. Select **Done editing**, then **Save**.
5. Reopen the workbook from the **Workbooks** list before making further changes. Saving repeatedly from an unsaved new workbook page creates separate copies.

To open it later, go to **Workbooks** in the Defender portal (security.microsoft.com/sentinel/workbooks) and select the workbook under **My workbooks**.

## Option 2 - Use the queries on their own

Paste any `.kql` file into **Advanced hunting** in the Defender portal and run it. The queries also work as a starting point for custom detection rules and your own workbooks.

Two queries need your Tier 0 host names. Edit the line marked `// EDIT`:

- `queries/part1-device-health/1-4-identity-servers.kql`
- `queries/part2-protection-controls/2-1-control-matrix.kql`

```kql
let names = dynamic(["dc01", "dc02", "adfs01", "ca01", "aadc01"]);  // EDIT
```

In the workbook, the same list comes from the **IdentityServers** parameter instead.

## Requirements

- Microsoft Defender for Endpoint, with devices onboarded
- Defender Vulnerability Management for the Part 2 queries (`DeviceTvm*` tables)
- Workbooks in the Defender portal - available to tenants using Microsoft Sentinel in the Defender portal, and to tenants on the Integrated Security Operations Center (ISOC) preview
- To create and edit the workbook: the **Workbooks** Unified RBAC permission with **Manage** access, or Security Administrator
- To view it: the **Workbooks** permission with **Reader** access, or Security Reader

## Notes

- **30-day look-back.** Device queries look back 30 days for each device's latest record and flag devices not seen for 7+ days, so devices that stop reporting are not silently dropped. Advanced hunting keeps 30 days of data.
- **Live data.** Device, alert and event data is near real time. The `DeviceTvm*` tables follow Defender Vulnerability Management's assessment cycle.
- **Control subcategories.** The Part 2 queries filter on `ConfigurationSubcategory`, which is case-sensitive. Tamper protection is a control within the **Antivirus** subcategory. Check the values in your tenant with:

  ```kql
  DeviceTvmSecureConfigurationAssessmentKB
  | where ConfigurationCategory == "Security controls"
  | summarize Controls = count() by ConfigurationSubcategory
  ```

- **Empty sections are findings.** If the ASR queries return nothing, check whether any ASR rules are configured - Part 2's control matrix will show them as non-compliant.

## Licence

Released under the [MIT Licence](LICENSE). Use, adapt and share freely; attribution is appreciated.

## Author

Seán O'Farrell - [cloud-secure.ai](https://cloud-secure.ai/)
