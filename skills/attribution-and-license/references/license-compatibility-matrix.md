# License Compatibility Matrix

A reference matrix for the `attribution-and-license` skill. Each cell answers: **if my project is licensed under TARGET, can I distill code from a source licensed under SOURCE?**

This matrix covers common cases. For combinations marked **UNKNOWN**, the skill asks the user to research and record a decision.

## Quick rules of thumb

- **Permissive sources (MIT, BSD-2/3, Apache-2.0, ISC)** are generally compatible with anything, including proprietary, as long as their attribution requirements are honored.
- **Weak copyleft (LGPL, MPL-2.0)** is generally compatible if the distilled code is kept in its own file(s) and not commingled.
- **Strong copyleft (GPL-2.0, GPL-3.0, AGPL-3.0)** forces the target project to adopt a compatible copyleft license, or to keep the distilled code at arm's length (rarely practical for porting).
- **Apache-2.0 → GPL-2.0** is **incompatible** in the GPL-2.0 direction due to the patent clause; **Apache-2.0 → GPL-3.0** is compatible.
- **AGPL-3.0** has additional network-use obligations. Treat as a strong copyleft with extra distribution requirements.

## Matrix

Rows are **source licenses**, columns are **target project licenses**.

| Source ↓ / Target → | MIT | BSD-3 | Apache-2.0 | LGPL-3.0 | MPL-2.0 | GPL-2.0 | GPL-3.0 | AGPL-3.0 | Proprietary |
| ------------------- | --- | ----- | ---------- | -------- | ------- | ------- | ------- | -------- | ----------- |
| **MIT**             | ✅   | ✅     | ✅          | ✅        | ✅       | ✅       | ✅       | ✅        | ✅           |
| **BSD-2-Clause**    | ✅   | ✅     | ✅          | ✅        | ✅       | ✅       | ✅       | ✅        | ✅           |
| **BSD-3-Clause**    | ✅   | ✅     | ✅          | ✅        | ✅       | ✅       | ✅       | ✅        | ✅           |
| **ISC**             | ✅   | ✅     | ✅          | ✅        | ✅       | ✅       | ✅       | ✅        | ✅           |
| **Apache-2.0**      | ✅   | ✅     | ✅          | ✅        | ✅       | ❌       | ✅       | ✅        | ✅           |
| **LGPL-3.0**        | ⚠️   | ⚠️     | ⚠️          | ✅        | ⚠️       | ⚠️       | ✅       | ✅        | ⚠️           |
| **MPL-2.0**         | ⚠️   | ⚠️     | ⚠️          | ⚠️        | ✅       | ⚠️       | ⚠️       | ⚠️        | ⚠️           |
| **GPL-2.0**         | ❌   | ❌     | ❌          | ❌        | ❌       | ✅       | ❌       | ❌        | ❌           |
| **GPL-2.0+**        | ❌   | ❌     | ❌          | ❌        | ❌       | ✅       | ✅       | ✅        | ❌           |
| **GPL-3.0**         | ❌   | ❌     | ❌          | ❌        | ❌       | ❌       | ✅       | ✅        | ❌           |
| **AGPL-3.0**        | ❌   | ❌     | ❌          | ❌        | ❌       | ❌       | ❌       | ✅        | ❌           |
| **Proprietary**     | ❌   | ❌     | ❌          | ❌        | ❌       | ❌       | ❌       | ❌        | ❌ (unless explicitly licensed) |

**Legend:**
- ✅ COMPATIBLE — proceed.
- ⚠️ COMPATIBLE WITH CONSTRAINTS — the distilled code typically must remain in its own file(s) and carry the source license. Treat as COMPATIBLE but record the constraint in the spec.
- ❌ INCOMPATIBLE — refuse unless the user explicitly overrides.

## Unlisted licenses

If either license is not in the matrix (e.g., CC-BY, custom OSS licenses, Elastic License, BSL, dual-licenses), classify as **UNKNOWN** and ask the user.

Examples that are typically problematic for distillation:

- **CC-BY / CC-BY-SA** — Creative Commons licenses are designed for content, not software. Avoid distilling code under these into a software project unless the user has done explicit due diligence.
- **Elastic License, BSL, SSPL** — source-available but not OSI-approved Open Source. Treat as **INCOMPATIBLE** with OSS targets unless the user demonstrates an explicit usage right.
- **No LICENSE file** — code is **all-rights-reserved by default** under most jurisdictions. Refuse to distill without written permission from the rights holder.

## Sources

This matrix summarizes commonly-accepted practice; it is not legal advice. The user is responsible for the final decision and for consulting counsel when stakes warrant.

Useful references the user can consult:
- SPDX License List: https://spdx.org/licenses/
- GNU License Compatibility list (for GPL-family details): https://www.gnu.org/licenses/license-list.html
- OSI-approved licenses: https://opensource.org/licenses/
