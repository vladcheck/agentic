---
name: get-note-links-from-text
description: Use when user asks you to get extract terms from text and turn them into Obsidian links.
license: MIT
user-invocable: true
version: "1.0.0"
metadata:
  author: valdemar_check <valdemarcheck@yandex.ru>
  deprecated: no
  tags: [obsidian]
---

# Get Note Links From Text

## Requirements

User gave you a plain text.

## Instructions

- Tell me what terms can I turn into note links, and how should I name these links. I name notes as concepts or ideas they describe. My notes are atomic (e.g. Hypervisor != Cloud hypervisor).

## Examples

Input: Multipass is a lightweight, cross-platform orchestration tool that provisions and manages on-demand, cloud-style Linux virtual machines (VMs) on a single host, abstracting the underlying hypervisor (KVM, Hyper-V, or QEMU) behind a unified CLI and GUI to deliver a streamlined local mini-cloud experience.

Output:

| Text fragment          | Suggested note title                               | Why                                           |
| ---------------------- | -------------------------------------------------- | --------------------------------------------- |
| Multipass              | [[Multipass]]                                      | Proper noun / specific tool                   |
| cross-platform         | [[Cross-platform software]]                        | Reusable property concept                     |
| orchestration tool     | [[Orchestration (computing)]]                      | The tool role is not atomic; orchestration is |
| provisions             | [[Provisioning (infrastructure)]]                  | Verb → noun concept                           |
| on-demand              | [[On-demand provisioning]]                         | Distinct provisioning pattern                 |
| cloud-style            | [[Cloud-style VM]] or [[Cloud-style provisioning]] | Only if you treat it as a distinct concept    |
| Linux                  | [[Linux]]                                          | Proper noun / OS                              |
| virtual machines (VMs) | [[Virtual machine]]                                | Core concept                                  |
| single host            | [[Host (virtualization)]]                          | “Host” is overloaded, so disambiguate         |
| hypervisor             | [[Hypervisor]]                                     | Core concept—not [[Cloud hypervisor]] here    |
| KVM                    | [[KVM]]                                            | Specific hypervisor                           |
| Hyper-V                | [[Hyper-V]]                                        | Specific hypervisor                           |
| QEMU                   | [[QEMU]]                                           | Specific hypervisor                           |
| CLI                    | [[CLI]]                                            | Core concept                                  |
| GUI                    | [[GUI]]                                            | Core concept                                  |
| local mini-cloud       | [[Local mini-cloud]]                               | Distinct composite concept                    |
