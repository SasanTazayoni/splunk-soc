# Apps vs Add-ons

The main guide already has the full breakdown in [Apps vs add-ons](../README.md#apps-vs-add-ons) (purpose, UI, examples, how they depend on each other, Splunkbase). These are the course's notes on top of that.

## App

- Has a **GUI / frontend** - each app brings its own front-end experience.
- **Resides on the Search Head** (the UI layer).
- Provides the experience you interact with: dashboards, reports, workflows.
- Examples: **AWS**, **Azure**, **Corelight**, **Splunk Enterprise Security**.

## Add-on

- Adds **functionality** to a Splunk instance - inputs, parsing, field extractions.
- **No GUI** - the "workstation view" doesn't change when you install one.
- Often one or more components **executing scripts** (e.g. scripted/modular inputs pulling from an API).
- Works behind the scenes to get data **in** and cleanly formatted.

## The CIM auto-mapping point

> "Any Splunk-supported app or add-on will map to the **CIM** automatically."

The **CIM (Common Information Model)** is Splunk's **standard field-naming schema** - it says, e.g., a source IP should be called `src_ip` no matter which vendor's log it came from (see the [glossary](../README.md#basic-terms-in-splunk-glossary)).

**Splunk-supported** (Splunk-built) add-ons are **CIM-compliant**: they automatically **normalise their fields to the CIM** as they parse data. That's what "map to the CIM automatically" means - you don't hand-map fields yourself.

**Why it matters:** premium apps like **Enterprise Security** run correlation searches written against **CIM field names**. Because a supported add-on has already normalised, say, Cisco and AWS logs to the same CIM fields, one detection works across **all** those sources at once - no per-source rewrites. This is the payoff of the _add-on normalises → app analyses_ relationship described in the main guide.
