# Driver migration tool

## Introduction

This capability addresses scenarios where you need to batch-migrate driver and tag configuration from third-party data collection platforms into files importable by EMQX Neuron. The online driver configuration migration service provided by EMQ can turn southbound connections and tags you have already configured in systems such as **KEPServerEX** and **Litmus Edge** into **EMQX Neuron**-importable JSON configuration files. When moving from KEPServerEX, Litmus Edge, and similar systems to Neuron, it significantly reduces the effort of re-entering devices, tags, and addresses in Neuron.

The tool is offered on the [**EMQ website**](https://www.emqx.com/en/products/emqx-neuron/migrator) as an **online service**; you do not need to install a separate application locally. Export configuration from the source platform per the documentation, upload it to the tool page, download the result after conversion, then complete import and integration testing on the Neuron side.

## Feature overview

| Capability | Description |
| ---------- | ----------- |
| Multiple sources | Select conversion logic by source type. Configuration exported from **KEPServerEX** and **Litmus Edge** is currently supported. |
| Protocol-level mapping | Maps drivers, devices, and tag/register models on each source to Neuron southbound devices and Group/Tag structure (see each source’s mapping tables in the documentation). |
| Reviewable results | After conversion, you can see summary information (e.g. device count, tag success/failure statistics); items that fail may be excluded from the final downloadable JSON, and failure reasons can be checked in the UI. |
| Neuron-oriented deliverable | Produces **EMQX Neuron**-importable configuration **JSON**, used in the data acquisition (southbound) section of the Neuron Dashboard via **Import**. |

::: tip

Supported industrial **protocols** and **limitations** (e.g. BCD, arrays, certificates) differ by source. Please verify each item in the **Supported conversion protocols** section of the corresponding migration guide.

:::

## When to use it

- **Platform replacement**: Your organization plans to use Neuron for acquisition previously handled by KEPServerEX or Litmus Edge, and you need to migrate live configuration in bulk.  
- **POC / pilot**: Validate Neuron’s connectivity and data quality with field devices in a small scope, and quickly align with existing tags and addressing.  
- **Disaster recovery and dual stack**: While keeping the original system, quickly reproduce an equivalent acquisition configuration in Neuron.  

## By source: overview and documentation

| Source product | Version and notes | Detailed guide |
| -------------- | ----------------- | -------------- |
| KEPServerEX | For **V6.0 and above**; export as **JSON** and other requirements are described in the guide. | [KEPServerEX to Neuron migration guide](./driver-migration-tool/kepware-to-neuron.md) |
| Litmus Edge | For **V4.0 and above**; steps such as exporting **Plain Text** from Device Management are in the guide. | [Litmus Edge to Neuron migration guide](./driver-migration-tool/litmus-edge-to-neuron.md) |

Both guides include: supported protocol lists, how to export files on the source system, upload and download steps on the website tool, behavior when **conversion fails**, Neuron-side **import and verification** recommendations, and protocol and data model mapping. Use the guide that matches the data you actually work with as your main reference.

## Prerequisites and notes

- **Network and reachability**: After import into Neuron, Neuron must be able to reach the **field devices** that the original acquisition targeted (IP, port, serial, etc. per your final Neuron configuration). Ensure network and security policies stay consistent with the migration plan.  
- **Unsupported items are skipped**: Unsupported protocols, out-of-range or incompatible addresses or data types may not appear in the final JSON. Check failed items in the tool UI, logs, or summary, then **add or adjust** manually in Neuron or re-convert after changing the source.  
- **Individual guides are authoritative**: Tool capabilities and protocol support evolve with releases; protocol lists, limitations, and screenshots in each **dedicated guide** are the source of truth.  

If you need extended support or run into issues during migration, refer to those guides together with **Neuron** official support channels to troubleshoot further.
