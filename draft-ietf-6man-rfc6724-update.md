---
title: Updating RFC 6724 based on Operational Experience
abbrev: RFC 6724 update
docname: draft-ietf-6man-rfc6724-update-05
cat: std
submissiontype: IETF
ipr: trust200902
area: Int
wg: 6MAN
kw: Internet-Draft
updates: 6724


author:
      -
        ins: N. Buraglio
        name: Nick Buraglio
        org: Energy Sciences Network
        email: buraglio@forwardingplane.net
      -
        ins: T. Chown
        name: Tim Chown
        org: Jisc
        email: Tim.Chown@jisc.ac.uk
      -
        ins: J. Duncan
        name: Jeremy Duncan
        org: Tachyon Dynamics
        email: jduncan@tachyondynamics.com

normative:
  RFC2119:
  RFC4193:
  RFC7078:
  RFC7526:

informative:
  RFC6724:
  RFC1918:
  RFC3484:
  RFC5220:
  RFC6555:
  RFC8305:
  RFC4861:
  RFC4191:
 
  

--- abstract

When {{RFC6724}} was published it defined an address selection algorithm along with a default policy table, and noted a number of examples where that policy table might benefit from adjustment for specific scenarios. It also noted that it is important for implementations to provide a way to change the default policies as more experience is gained. This update draws on several years of operational experience to refine RFC 6724 further, with particular emphasis on preference for the use of ULA addresses over IPv4 addresses, mandatory support for Rule 5.5, and requirements both that sites apply the ULA filtering principles described in RFC 4193 and that ULAs are not published in the global DNS. This update also demotes the preference for 6to4 addresses. The changes to default behavior improve supportability of common use cases, including automatic / unmanaged scenarios. It is recognized that some less common deployment scenarios may require explicit configuration or custom changes to achieve desired operational parameters.
  
--- middle

# Introduction

Since its publication in 2012, {{RFC6724}} has become an important mechanism by which nodes can perform address selection, deriving the most appropriate source and destination address pair to use from a
candidate set by following the procedures defined in the RFC. Part of the process involves the use of a policy table, where the precedence and labels for address prefixes are listed, and for which a default table is defined.

It was always expected that the default policy table may need to be updated from operational experience; section 2.1 says "It is important that implementations provide a way to change the default policies as more experience is gained" and points to the examples in Section 10, which include Section 10.6 where a ULA example is presented.

This document is written on the basis of such operational experience, in particular for scenarios where ULAs are used for their intended purpose within a site, but also includes updated requirements on support for RFC 6724 Rule 5.5, and for filtering packets with ULA source addresses at site borders. 

An IPv6 deployment, whether enterprise, residential or other, may use combinations of IPv6 GUAs, IPv6 ULAs, IPv4 globals, IPv4 RFC 1918 addressing, and may or may not use some form of NAT. However, this document makes no comment or recommendation on how ULAs are used, or on the use of NAT in an IPv6 network. 


# Terminology

{::boilerplate bcp14-tagged}


# Operational Issues Regarding Preference for IPv4 addresses over ULAs

Where getaddrinfo() or a comparable API is used, the sorting behavior should take into account both
the source address of the requesting node as well as the destination addresses returned, and sort these following the procedures defined in RFC 6724.

The current default policy table leads to preference for IPv6 GUAs over IPv4 globals, which is widely considered preferential behavior to support greater use of IPv6 in dual-stack environments. This allows sites to phase out IPv4 as its evidenced use becomes ever lower.

However, the default policy table also puts IPv6 ULAs below all IPv4 addresses, including {{RFC1918}} addresses. For many site operators this behavior will be counter-intuitive, and may create difficulties with respect to planning, operational, and security implications for environments where ULA addressing is used in IPv4/IPv6 dual-stack network scenarios. The expected prioritization of IPv6 traffic over IPv4 by default, as happens with IPv6 GUA addressing, does not by default happen for ULAs.

As a result, the use of ULAs is not a viable option for dual-stack networking transition planning, large scale network modeling, network lab environments or other modes of large scale networking that run both IPv4 and IPv6 concurrently with the expectation that IPv6 will be preferred by default.

This document updates the default policy table to elevate the preference for ULAs such that ULAs will be preferred over all IPv4 addresses, providing more consistent and less confusing behavior for operators, and to assist operators in phasing out IPv4 from dual-stack environments, since by this update IPv6 GUAs and ULAs will be preferred over any IPv4 addresses. This is an important enabler for sites seeking to move from dual-stack to IPv6-only networking.

This change aims to improve the default handling of address selection for common cases, and unmanaged / automatic scenarios rather than those where DHCPv6 is deployed. Sites using DHCPv6 for host configuration management can make use of implementations of {{RFC7078}} to apply changes to the {{RFC6724}} policy table.

The changes are discussed in more detail in the following sections, with a further section providing a summary of the proposed updates.

# Preference of 6to4 addresses

The anycast prefix for 6to4 relays was deprecated by {{RFC7526}} in 2015, and since that time the use of 6to4 addressing has further declined to the point where it is generally not seen and can be considered to all intents and purposes deprecated in use.  

This document therefore demotes the precedence of the 6to4 prefix in the policy table to the same minimum preference as carried by the deprecated site local and 6bone address prefixes.

# Adjustments to RFC 6724

This update makes three specific changes to RFC 6724: first, to update the default policy table, second, change the next hop advertised prefix by the next hop (Rule 5.5) to a MUST, and third the (recommended) option to automatically insert "known-local" ULA prefixes into the policy table.

## Policy Table Update

This update alters the default policy table listed in Rule 2.1 of RFC 6724.

The table below reflects the current RFC 6724 state on the left, and the updated state defined by this RFC on the right:

~~~~~~~~~~
                    RFC 6724                                            Updated                  
      Prefix        Precedence Label                      Prefix        Precedence Label              
      ::1/128               50     0                      ::1/128               50     0
      ::/0                  40     1                      ::/0                  40     1
      ::ffff:0:0/96         35     4                      ::ffff:0:0/96         20     4 (*)
      2002::/16             30     2                      2002::/16              5     2 (*)
      2001::/32              5     5                      2001::/32              5     5
      fc00::/7               3    13                      fc00::/7              30    13 (*)
      ::/96                  1     3                      ::/96                  1     3
      fec0::/10              1    11                      fec0::/10              1     11
      3ffe::/16              1    12                      3ffe::/16              1     12

 (*) value(s) changed in update

~~~~~~~~~~

This preference table update moves 2002::/16 to de-preference its status in line with RFC 7526 and changes the precedence of fc00::/7 above legacy IPv4, with ::ffff:0:0/96 now set to precedence 20.

## Rule 5.5 

The heuristic for address selection defined in Section 5.5 of RFC 6724 to prefer addresses in a prefix advertised by a next-hop router has proven to be very useful. RFC 6724 does not state any requirement for SHOULD or MUST for this heuristic to be used; this update therefore amends section 5.5 to reflect that a system MUST apply the next-hop tracking heuristic, comminly referred to as Rule 5.5.

## Automatic insertion of Known-Local ULAs

** Do we include this at all, or beef up the RFC 4193 section 4.4 text that says "AAAA and PTR records for locally assigned local IPv6 addresses are not recommended to be installed in the global DNS", and that this *IS* a misconfiguration, and point to Dan Wing's data that suggests most people don't do it?

This update also introduces the concept of "known-local" ULAs. The new default behaviour introduced by this update elevates all ULAs above all IPv4 addresses. However, it is possible that a node in a destination network may have a host name with a ULA address in its global DNS entry. In such cases connectivity to that node is unlikely to work (in the absence of specific configured routing).

** Do we want to add support for misconfigurations?  Data suggests that less than 0.1% of published AAAAs are ULAs

To avoid such potential issues, this update additionally states that an implementation SHOULD automatically add rows in the default policy table for all ULA site prefixes that can be derived from received Router Advertisements (RAs) {{RFC4861}} given their Prefix Information Options (PIOs) or Route Information Options (RIOs) {{RFC4191}}. Such derived /48 prefixes are referred to as "known-local" ULAs and SHOULD have a precedence of 45. 

Note that RFC 6724 states in Section 2.1 that "An implementation MAY automatically add additional site-specific rows to the default table based on its configured addresses, such as for Unique Local Addresses (ULAs)" and in Section 10.3 that "Since ULAs are defined to have a /48 site prefix, an implementation might choose to add such a row automatically on a machine with a ULA." Here we are elevating that MAY/might to a SHOULD.

Nodes implementing the automated increased precedence for known-local ULAs /48s SHOULD also set the default precedence for the ULA label (fc00::/7) to 10. 

** Note ** the downside to this is that if a node cannot determine that another ULA in the same organisational network is "known-local" it will prefer IPv4 if dual-stack 

# Configuration of the default policy table

As stated in Section 2.1 of RFC 6724 "IPv6 implementations SHOULD support configurable address selection via a mechanism at least as powerful as the policy tables defined here". While this document defines changes to RFC 6724 behavior based on operational experience to date, it is important that node policy tables can be changed once deployed to support future emerging use cases.

# Intended behaviors

In this section we reiew the intended default behaviors after this update is applied.

## GUA-GUA preferred over IPv4-IPv4

This is the current behaviour, and remains unaltered. The rationale is to promote use of IPv6 GUAs in dual-stack environments.

## ULA-ULA preferred over GUA-GUA

This is the current behaviour, and remains unaltered. The rationale is to support consistent, stable addressing in a site where the global prefix might change over time, or become unavailable.

## ULA-ULA preferred over IPv4-IPv4

This is a change introduced by this update. RFC 6724 as orginally defined would lead to IPv4 being preferred over ULAs, which is contrary to the spirit of the GUA preference over IPv4, and to the goal of removing use of IPv4 in a dual-stack site before transitioning to IPv6-only.

## IPv4-IPv4 preferred over ULA-GUA

An IPv6 ULA address will only be preferred over an IPv4 address if both IPv6 ULA source and destination addresses are available. With Rule 5 of Section 6 of RFC 6724 and the ULA-specific label added in RFC 6724 (which was not present in RFC 3484) an IPv4 source and destination will be preferred over an IPv6 ULA source and an IPv6 GUA destination address, even though generally IPv6 ULA addresses are preferred over IPv4 in the policy table as proposed in this update. The IPv4 matching label trumps ULA-GUA.

# Discussion of ULA source with GUA or remote ULA destinations

A potential problem exists when a ULA source attempts to communicate with GUA or remote ULA destinations. In these scenarios, the ULA source is generally intended for communication only with the local network, meaning an individual site, several sites that are part of the same organization, or multiple sites across cooperating organizations, as detailed in RFC 4193. As a result, most GUA and ULA destinations are not attached to the same local network as the ULA source and are, therefore, not reachable from the ULA source.

When only a ULA source is available for communication with GUA destinations, this generally implies no connectivity to the IPv6 Internet is available. Otherwise, a GUA source would have been made available and selected for use with GUA destinations. As a result, the ULA source will typically fail when it attempts to communicate with most GUA destinations. However, corner cases exist where the ULA source will not fail, such as when GUA destinations are attached to the same local network as the ULA source.

Receiving a DNS response for a ULA destination that is not attached to the local network, in other words, a remote ULA destination, is considered a misconfiguration in most cases, or at least this contradicts the operational guidelines provided in Section 4.4 of RFC 4193. Nevertheless, this can occur, and the ULA source will typically fail when it attempts to communicate with ULA destinations that are not attached to the same local network as the ULA source.

This section discusses several complementary mechanisms involved with these scenarios.

## The ULA Label and its Precedence

RFC 6724 added (in obsoleting RFC 3484) a separate label for ULA (fc00::/7), whose default precedence is raised by this update. This separate label interacts with Rule 5 of Section 6 of RFC 6724, which says;

      Rule 5: Prefer matching label.
      If Label(Source(DA)) = Label(DA) and Label(Source(DB)) <> Label(DB), then prefer DA.  Similarly, if       Label(Source(DA)) <> Label(DA) and Label(Source(DB)) = Label(DB), then prefer DB.

The ULA source label will not match the GUA destination label in the first scenario. Therefore, an IPv4 destination, if available, will be preferred over a GUA destination with a ULA source, even though the GUA destination has higher precedence than the IPv4 destination in the policy table. This means the IPv4 destination will be moved up in the list of destinations over the GUA destination with the ULA source. 

If the ULA (fc00::/7) label is removed from the policy table, a GUA destination with a ULA source will be preferred over an IPv4 destination, as GUA and ULA will be part of the same label (::/0).

The ULA source label will match the ULA destination label in the second scenario; therefore, whether part of the local network or not, a ULA destination will be preferred over an IPv4 destination.

If the ULA label (fc00::/7) has its precedence lowered below IPv4 or the IPv4 precedence is raised above ULA, as with the RFC 6724 unupdate policy table. In this case, an IPv4 destination will be preferred over all ULA destinations. 

## Happy Eyeballs

Regardless of the preference resulting from the above discussion, Happy Eyeballs version 1 (RFC 6555) or version 2 (RFC 8305), if implemented, will try both the GUA or ULA destination with the ULA source and the IPv4 destination and source pairings. The ULA source will typically fail to communicate with most GUA or remote ULA destinations, and IPv4 will be preferred if IPv4 connectivity is available unless the GUA or ULA destinations are attached to the same local network as the ULA source.

## Try the Next Address

As stated in Section 2 of RFC 6724,

      Well-behaved applications SHOULD NOT simply use the first address returned from an API such as
      getaddrinfo() and then give up if it fails. For many applications, it is appropriate to iterate 
      through the list of addresses returned from getaddrinfo() until a working address is found. For
      other applications, it might be appropriate to try multiple addresses in parallel (e.g., with some
      small delay in between) and use the first one to succeed.

Therefore, when an IPv4 destination is preferred over GUA or ULA destinations, IPv4 will likely succeed if IPv4 connectivity is available, and the GUA or ULA destination may only be tried if Happy Eyeballs is implemented. 

On the other hand, if the GUA or ULA destination with the ULA source is preferred, the ULA source will typically fail to communicate GUA or ULA destinations that are not connected to the same local network as the ULA source. However, suppose the operational guidelines in Section 4.3 of RFC 4193 are followed, as discussed in the Security Considerations below. In this case, recognizing this failure can be accelerated, and transport layer timeouts (e.g., TCP) can be avoided. These guidelines will cause a Destination Unreachable ICMPv6 Error to be received by the source device, signaling the next address in the list to be tried, as discussed above.

# Filtering ULA-source addresses at site borders

RFC 4193 Section 4.3 states "Site border routers and firewalls should be configured to not forward
any packets with Local IPv6 source or destination addresses outside of the site, unless they have been explicitly configured with routing information about specific /48 or longer Local IPv6 prefixes".

This document requires that such site border routers and firewalls MUST be configured to not forward
by default any packets with Local IPv6 source or destination addresses outside of the site, unless specifically configured to do so.

# ULA addresses in the gloabl DNS

Section 4.3 of RFC 4193 states that "At the present time, AAAA and PTR records for locally assigned local IPv6 addresses are not recommended to be installed in the global DNS."

This document requires that AAAA and PTR records for ULAs MUST NOT be installed in the global DNS.

# The practicalities of implementing address selection support

As with most adjustments to standards, and using the introduction of RFC 6724 as a measuring stick, the updates defined in this document will likely take several years to become common enough for consistent behavior within most operating systems. At the time of writing, it has been over 10 years since RFC 6724 has been published but we continue to see existing commercial and open source operating systems exhibiting RFC 3484 behavior.

While it should be noted that RFC 6724 defines a solution to adjust the address preference selection table that is functional theoretically, operationally the solution is operating system dependent and in practice policy table changes cannot be signaled by any currently deployed network mechanism. While RFC 7078 defines such a DHCPv6 option, it is not widely implemented. This lack of an intra-protocol or network-based ability to adjust address selection preference, along with the inability to adjust a notable number of operating systems either programmatically or manually, renders operational scalability of such a mechanism challenging.  

It is especially important to note this behavior in the long lifecycle equipment that exists in industrial control and operational technology environments due to their very long mean time to replacement/lifecycle.

# Limitations of RFC 6724

The procedures defined in RFC 6724 do not give optimal results for all scenarios. The aim of this update is to improve the behavior for the most common scenarios.

It is widely recognised in the IETF 6man WG that the whole 3484/6724/getaddrinfo() model is fundamentally inadequate for optimal address selection.  A model that considered adress pairs directly, rather than sorting on destination addresses with the best source for that address, would be preferable, but beyond the scope of this document.

To simplify address selection, operators may instead look to deploy IPv6-only, and may choose to only use GUA addresses and no ULA addresses.

# Implementation note

To be removed on publication.

** Add text - Jeremy's work.  Other?

# Acknowledgements 

The authors would like to acknowledge the valuable input and contributions of the 6man WG including Brian Carpenter, XiPeng Xiao, Eduard Vasilenko, David Farmer, Bob Hinden, Ed Horley, Tom Coffeen, Scott Hogg, Chris Cummings, Paul Wefel, Dale Carder, Erik Auerswald, Ole Troan, Eric Vyncke, Timothy Winters, Kyle Rose, Lorenzo Colitti, Jen Linkova, and Mark Smith.

# Security Considerations

There are no direct security considerations in this document.

The mixed preference for IPv6 over IPv4 from the default policy table in RFC 6724 represents a potential security issue, given an operator may expect ULAs to be used when in practice RFC 1918 addresses are used instead.

Operators should be mindful of cases where one node is compliant with RFC 6724 as originally published and another node is compliant with the update presented in this document, as there may be inconsistent behaviour for communications initiated in each direction. Similarly, differences between current RFC 6724-compliant and RFC 3484-compliant nodes may also be observed. Ultimately all nodes should be made compliant to the updated specification described in this document.

# IANA Considerations

None.

# Appendix A. Changes since RFC6724

* Update to default preference table moving 6to4 address block 2002::/16 to de-preference status in line with {{RFC7526}},
* Change the default address selection to move fc00::/7 to preference 30, above legacy IPv4,  
* Change ::ffff:0:0/96 to preference 20,
* Add section relating to Happy Eyeballs,
* Change section 5.5 to require interface prefix tracking.
* Note added for RFC 4193 ULA filtering as a MUST.
* Note added on RFC 4193 ULAs in the Global DNS as a MUST NOT.

--- back
