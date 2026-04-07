## Overview
This directory contains curated Suricata rules for pfSense focused on detecting Man-in-the-Middle (MiTM) attacks and multi-stage data exfiltration. Rulesets include detailed detections, cross-flow correlation via xbits, and tuning guidance for deployment on pfSense Plus with Suricata 7.x. There rules are built for homelab and personal usages only.

## Documentation Links
- [Suricata Rules: MiTM Attacks (9000000)](rules/9000000-mitm-attacks.rules)
- [Suricata Rules: Data Exfiltration (8000000)](rules/8000000-data-exfiltration-detection.rules)
- [Coding Instructions (Suricata)](../../docs/instructions/suricata.coding.instructions.md)
- [Requirements](../../docs/requirements/suricata.requirements.instruction.md)
- [User Guides](../README.md)

## Prerequisites
- pfSense Plus (recommended) or pfSense CE with Suricata package
- Suricata 7.0+ (rules use `xbits`, advanced dns/tls keywords, and app-layer parsers)
- Administrative access to the pfSense Suricata GUI or suricata.yaml for enabling app-layer ARP, DNS and protocol parsers
- Sufficient RAM/CPU for IDS/IPS throughput depending on interface speeds

## Installation
1. Copy the rules files into the `pfsense/suricata/rules/` directory on your rules repository.
2. On pfSense Suricata GUI, add these files into the Suricata rules configuration or include via suricata.yaml `-S` include path.
3. Validate rules:
```bash
suricata -T -c /etc/suricata/suricata.yaml -S /etc/suricata/rules/9000000-mitm-attacks.rules -S /etc/suricata/rules/8000000-data-exfiltration-detection.rules
```
4. Restart Suricata or reload rules from the GUI.

## Configuration
1. Configuration file locations
	- Suricata config: `/etc/suricata/suricata.yaml`
	- Rules directory: `/etc/suricata/rules/`
2. Environment variables
	- Ensure `HOME_NET` and `EXTERNAL_NET` are correctly defined in your Suricata configuration
3. External service dependencies
	- Access to DNS, TLS inspection logs, and network time synchronization (NTP)
4. Database setup
	- Optional: External EVE/ELK or SIEM to consume Suricata EVE JSON output for long-term correlation

## Development Workflow
- Branch strategy: use feature branches for rule updates (e.g., `rules/9000xxx-description`)
- Code review: submit PRs with test pcap or sample alerts demonstrating detection
- Testing: validate rules locally with `suricata -T` and replay pcaps with `suricata -r`
- CI/CD: integrate rule validation in CI to run Suricata tests and linting

## Build Instructions
No build required. Rules are plain text files; validate with `suricata -T` after changes.

## Testing
Run Suricata in test mode or replay pcaps that exercise rule signatures.

Example:
```bash
suricata -T -c /etc/suricata/suricata.yaml -S /etc/suricata/rules/9000000-mitm-attacks.rules
suricata -r samples/mitm_arpspoof.pcap -S /etc/suricata/rules/9000000-mitm-attacks.rules -l /tmp/suricata-test
```

## Deployment
1. Staging: Deploy rules in detection-only mode (IDS) and monitor false positives for 72 hours.
2. Production: Enable inline blocking (IPS) after tuning thresholds and whitelisting known infrastructure.
3. Rollback: Revert to prior ruleset version via configuration management (Git) and reload Suricata.
4. Monitoring: Ship EVE JSON to SIEM and create dashboards for `alert` and `drop` events.

## Security Considerations
- Ensure Suricata runs with least privilege and logs are forwarded securely.
- Protect rule files and Git repository with access controls to prevent tampering.
- Regularly review and tune rules to reduce false positives and avoid blocking legitimate traffic.

## Performance Considerations
- Use fast_pattern where available (rules already include fast_pattern hints).
- Apply rule sets to specific interfaces and use capture filters to limit processed traffic.
- Monitor CPU, memory, and packet drop metrics; move heavy rules to detection-only where necessary.

## Troubleshooting
| Issue   | Solution   |
| ------- | ---------- |
| Rules fail validation | Run `suricata -T` to get parse errors and correct rule syntax |
| Excessive false positives | Run in IDS mode, add suppress rules for known hosts, adjust thresholds |
| High CPU usage | Disable expensive rules or run them only in detection mode |

## Monitoring and Logging
- Metrics: Suricata EVE JSON (alerts, dns, http, tls), stats.log for performance
- Log locations: `/var/log/suricata/eve.json`, `/var/log/suricata/stats.log`
- Alerting: Integrate with SIEM (Splunk/ELK) to trigger on high-confidence correlations (vpn+exfil)
- Dashboards: Create visualizations for `xbits` setters and high-severity SIDs

## Maintenance
- Schedule: Review and test rules quarterly or when Suricata major versions change
- Backups: Store rules and suricata.yaml in Git with signed commits
- DR plan: Keep last-known-good rulesets; rollback via config management
- Support: Network security team (see Contact Information)

## Contributing
1. Code style: Follow Suricata rule writing conventions and include fast_pattern/threshold where possible
2. Pull requests: Include sample pcaps and `suricata -T` output showing rule triggers
3. Issues: Report false positives with pcap and timestamp for triage

## License
Rules provided under MIT license. See `LICENSE` in repository root for details.

## Acknowledgements
Credits to the original authors of the rules and public threat research (MITRE ATT&CK, NIST).

## Contact Information
- Primary maintainer: Network Security Team <security@example.org>
- Development team: sec-dev@example.org
- Support channel: #netsec on internal Slack

## Change Log
| Version | Date       | Changes         |
| ------- | ---------- | --------------- |
| 1.0.0   | 2026-04-07 | Initial release of MiTM & Data Exfil rules |

_Last updated: 2026-04-07_
_Personal Homelab - 2026_
