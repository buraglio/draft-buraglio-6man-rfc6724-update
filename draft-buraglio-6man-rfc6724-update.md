---
title: Preference for ULAs over RFC1918 addresses in RFC6724
abbrev: Prefer ULAs over RFC1918 addresses
docname: draft-buraglio-6man-rfc6724-update-00
cat: std
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
 
  

--- abstract

This document updates RFC 6724 based on operational experience gained since its publication over ten years ago. In particular it updates the preference of Unique Local Addresses (ULAs) in the default address selection policy table, which as originally defined by RFC 6724 has lower precedence than legacy IPv4 addressing. The update places both IPv6 Global Unicast Addresses (GUAs) and ULAs ahead of all IPv4 addresses on the policy table to better suit operational deployment and management of ULAs in production. This document also updates requirements on configurability of the policy table and preference for using ddresses from a prefix advertised by a next-hop router, and demotes the preference for 6to4 addresses in the default policy table. These changes to default behavior improve supportability of common use cases such as, but not limited to, automatic / unmanaged scenarios. It is recognized that some less common deployment situations may require explicit confioguration or custom changes to acheive desired operational parameters.  

--- middle

# Introduction

When {{RFC6724}} was published in 2012 it was expected that the default policy table may need to be updated from operational experience; section 2.1 says "It is important that implementations provide a way to change the default policies as more experience is gained" and points to the examples in Section 10, including Section 10.6 which considers a ULA example.

This document is written on the basis of operational experience, in particular for scenarios where ULAs are used within a site. The current default policy table in RFC 6724 leads to preference for IPv6 GUAs over IPv4 globals, which is widely considered to be preferential behaviour to support greater use of IPv6 in dual-stack environments, and to allow sites to phase out IPv4 as its use becomes ever lower.

However, the  default policy table also puts IPv6 ULAs below all IPv4 addresses, including {{RFC1918}} addresses. For many site operators this behavior will be counter-intuitive, and may create difficulties with respect to planning, operational, and security implications for environments where ULA addressing is used in certain IPv4/IPv6 dual-stack network scenarios. The expected prioritization of IPv6 traffic over IPv4 by default, as happens with IPv6 GUA addressing, will not happen for ULAs.

An IPv6 deployment, whether enterprise, residential or other, may use combinations of IPv6 GUAs, IPv6 ULAs, IPv4 globals, IPv4 RFC 1918 addressing, and may or may not use some form of NAT. This document makes no comment or recommendation on how ULAs are used, or on NAT, but notes that operationally where GUAs and ULAs are used alongside RFC 1918 addressing, an IPv6 GUA would be selected to reach an IPv6 GUA destination, but where only ULAs and RFC1918 addressing are used, RFC 1918 addresses will be preferred.

This document updates the default policy table to elevate the preference for ULAs such that ULAs will be preferred over all IPv4 addresses, providing more consistent and less confusing behaviour for operators.

Note that ULAs must never be used across site boundaries as if they were under unregistered global prefixes. Nothing in this document pertains to such misuse. 

The emergence of this issue also reinforces the need for the original RFC 6724 address slection policy table to be configurable. RFC 6724 Section 2.1 states that the table SHOULD be configurable; this document proposes elevating that requirement to MUST, to ensure that any device can have its policy table tuned for the scenario in which it is deployed. Section 10 of RFC 6724 gives other examples of why configurability is important. 

This document aims to improve the default handling of address selection for common cases, and unmanaged / automatic scenarios rather than those where DHCPv6 is deployed. Sites using DHCPv6 for host configuration management can make use of implementations of {{RFC7078}} to apply changes to the RFC 6724 policy table.

It has also become clearer from operational experience that the heuristic to prefer addresses drawn from a prefix advertised by a next-hop router is a valuable one to use.  This text therefore also proposes elevating that requirement in Section 5.5 from SHOULD to MUST.

These updates are discussed in more detail in the following sections, with a further section providing a summary of the proposed updates.

Authors' note for the -00 version: this draft also captures some of the meta discussion around not only the proposed changes but other suggestions drawn from 6man WG list discussions.  These elements will be removed if there is consensus to move the document forward on the proposed path.

# Terminology

{::boilerplate bcp14-tagged}


# Unintended Operational Issues Regarding IPv6 Preference and ULAs

The preference for use of IPv6 addressing over IPv4 addressing in {{RFC6724}} is inconsistent. As written, RFC 6724 section 10.3 states:

~~~~~~~~~~
"The default policy table gives IPv6 addresses higher precedence than
IPv4 addresses.  This means that applications will use IPv6 in
preference to IPv4 when the two are equally suitable.  An
administrator can change the policy table to prefer IPv4 addresses by
giving the ::ffff:0.0.0.0/96 prefix a higher precedence".
~~~~~~~~~~

The expected behavior would be that ULA address space would be preferred over legacy IPv4, however this is not the case. This presents an issue with any environment that will use ULA addressing alongside legacy IPv4, whether global or RFC 1918. This is counter to the standard expectations for legacy IPv4 / IPv6 dual-stack behavior in preferring IPv6, which is the case for GUA addressing.

## Operational Implications

There are demonstrated and easily repeatable uses cases of ULA not being preferred in some OS and network equipment over legacy IPv4 that necessitate an update to RFC 6724 to better reflect the original intent of the RFC. 

Below is an example of a gai.conf file from a modern Linux installation as of 25 May 2023:

~~~~~~~~~~
# Configuration for getaddrinfo(3).
#
# So far only configuration for the destination address sorting is needed.
# RFC 3484 governs the sorting.  But the RFC also says that system
# administrators should be able to overwrite the defaults.  This can be
# achieved here.
#
# All lines have an initial identifier specifying the option followed by
# up to two values.  Information specified in this file replaces the
# default information.  Complete absence of data of one kind causes the
# appropriate default information to be used.  The supported commands include:
#
# reload  <yes|no>
#    If set to yes, each getaddrinfo(3) call will check whether this file
#    changed and if necessary reload.  This option should not really be
#    used.  There are possible runtime problems.  The default is no.
#
# label   <mask>   <value>
#    Add another rule to the RFC 3484 label table.  See section 2.1 in
#    RFC 3484.  The default is:
#
#label ::1/128       0
#label ::/0          1
#label 2002::/16     2
#label ::/96         3
#label ::ffff:0:0/96 4
#label fec0::/10     5
#label fc00::/7      6
#label 2001:0::/32   7
#
#    This default differs from the tables given in RFC 3484 by handling
#    (now obsolete) site-local IPv6 addresses and Unique Local Addresses.
#    The reason for this difference is that these addresses are never
#    NATed while IPv4 site-local addresses most probably are.  Given
#    the precedence of IPv6 over IPv4 (see below) on machines having only
#    site-local IPv4 and IPv6 addresses a lookup for a global address would
#    see the IPv6 be preferred.  The result is a long delay because the
#    site-local IPv6 addresses cannot be used while the IPv4 address is
#    (at least for the foreseeable future) NATed.  We also treat Teredo
#    tunnels special.
#
# precedence  <mask>   <value>
#    Add another rule to the RFC 3484 precedence table.  See section 2.1
#    and 10.3 in RFC 3484.  The default is:
#
#precedence  ::1/128       50
#precedence  ::/0          40
#precedence  2002::/16     30
#precedence ::/96          20
#precedence ::ffff:0:0/96  10
#
#    For sites which prefer IPv4 connections change the last line to
#
#precedence ::ffff:0:0/96  100

#
# scopev4  <mask>  <value>
#    Add another rule to the RFC 6724 scope table for IPv4 addresses.
#    By default the scope IDs described in section 3.2 in RFC 6724 are
#    used.  Changing these defaults should hardly ever be necessary.
#    The defaults are equivalent to:
#
#scopev4 ::ffff:169.254.0.0/112  2
#scopev4 ::ffff:127.0.0.0/104    2
#scopev4 ::ffff:0.0.0.0/96       14
~~~~~~~~~~

The legacy IPv4 address range in the gai.conf file is "scopev4" and the prefix ::ffff:0.0.0.0/96 which has a higher precedence (35) in RFC 6724 than the ULA prefix of fc00::/7 (3). This results in legacy IPv4 being preferred over IPv6 ULA. While not inherently undesirable, the operational outcome when utilizing dual-stack with ULA is inconsistent and imparts unnecessary difficulty for both troubleshooting and creating the requisite baseline of the expected behavior which are both requirements for supportable production deployments. Depending on the host implementation, security baseline expectations can be inconsistent at best and haphazard at worst.

As the gai.conf file, or an equivalent within a given operating system, is referenced it dictates the
behavior of the getaddrinfo() or analogous process. More specifically, where getaddrinfo() or a comparable API is used, the sorting behavior should take into account both
the source address of the requesting host as well as the destination addresses returned and sort according to both source and destination addresses, i.e, when a ULA address is
returned, the source address selection should return and use a ULA address if available. Similarly, if a GUA address is returned the source address selection should return a GUA source address if available.

However, there are clearly evidenced example of three failure scenarios:

1. ULA per RFC 6724 is less preferred (the Precedence value is lower) than all legacy IPv4 (represented by ::ffff:0:0/96 in the aforementioned table).
2. Because of the lower Precedence value of fc00::/7, if a host has legacy IPv4 enabled, it will use legacy IPv4 before using ULA.
3. A dual-stacked client will source the traffic from the legacy IPv4 address, meaning it will require a corresponding legacy IPv4 destination address.

For scenario number 3, when a host resolves through DNS a destination with A and AAAA DNS records, the host will choose the A record to get an legacy IPv4 address for the destination, meaning ULA IPv6 is rendered unused. 

As a result, the use of ULAs is not a viable option for dual-stack networking transition planning, large scale network modeling, network lab environments or other modes of large scale networking that run both IPv4 and IPv6 concurrently with the expectation that IPv6 will be preferred by default.

# Configurability of the Default Policy Table

In principle the above problem would not be an issue were the RFC 6724 default policy table readily configurable in all systems. Section 10.6 states that ULAs can be preferred by adding a site-specific entry to the default policy table. In practice, this is currently not always possible.

While conceptually the intent was for a configurable, longest-match table to be adjusted as needed. In practice, modifying the prefix policy table remains difficult across platforms, and in some cases impossible. Embedded, proprietary, closed source, and IoT devices are especially difficult to adjust, and are in many cases incapable of any adjustment. Large scale manipulation of the policy table also remains out of the realm of realistic support for small and medium scale operators due to lack of ability to manipulate all the hosts and systems, or a lack of tooling and access.

Operational experience suggests that the default policy table needs to be as configurable as possible in as many systems as possible. This update therefore proposes that the requirement that IPv6 implementations support configurable address selection via a mechanism at least as powerful as the policy table be elevated from a SHOULD to a MUST.

Authors' note for the -00 version: of course we state above that for some platforms, the ability to implement such a method is challenging.  The question for the 6man WG is how to ensure configurability is as widespread as possible.

# Next-hop router heuristic

The heuristic for address selection defined in Section 5.5 of RFC 6724 to prefer addresses in a prefix advertised by a next-hop router has proven to be very useful.  RFC 6724 does not state any requirement for SHOULD or MUST for this heuristic to be used; this update therefore proposes stating that the application of the heuristic be a MUST.

# Preference of 6to4 addresses

The anycast prefix for 6to4 relays was deprecated by {{RFC7526}} in 2015, and since that time the use of 6to4 addressing has further declined to the point where it is generally not seen and can be considered to all intents and purposes deprecated in use.  This document therefore demotes the preference of the 6to4 prefix in the policy table to the same minimum preference as carried by the deprecated site local and 6bone address prefixes.

# Adjustments to RFC 6724

Rule 2.1 of RFC 6724 states:

~~~~~~~~~~
IPv6 implementations SHOULD support configurable address selection
via a mechanism at least as powerful as the policy tables defined
here.  It is important that implementations provide a way to change
the default policies as more experience is gained.  Sections 10.3
through 10.7 provide examples of the kind of changes that might be
needed.
~~~~~~~~~~

This document updates RFC 6724 section 2.1 to the following:

~~~~~~~~~~
IPv6 implementations MUST support configurable address selection
via a mechanism at least as powerful as the policy tables defined
here.  It is important that implementations provide a way to change
the default policies to ensure operational supportability and flexibility in deployment.
Sections 10.3 through 10.7 provide examples of the kind of changes that might be
needed.
~~~~~~~~~~

Rule 2.1 of RFC 6724 further states:

~~~~~~~~~~
If an implementation is not configurable or has not been configured,
   then it SHOULD operate according to the algorithms specified here in
   conjunction with the following default policy table:


      Prefix        Precedence Label
      ::1/128               50     0
      ::/0                  40     1
      ::ffff:0:0/96         35     4
      2002::/16             30     2
      2001::/32              5     5
      fc00::/7               3    13
      ::/96                  1     3
      fec0::/10              1    11
      3ffe::/16              1    12
~~~~~~~~~~

This document updates RFC 6724 section 2.1 to the following:

~~~~~~~~~~
If an implementation is not configurable or has not been configured,
   then it SHOULD operate according to the algorithms specified here in
   conjunction with the following default policy table:


      Prefix        Precedence Label
      ::1/128               50     0
      ::/0                  40     1
      fc00::/7              35    13
      ::ffff:0:0/96         30     4
      2001::/32              5     5
      2002::/16              1     2
      ::/96                  1     3
      fec0::/10              1    11
      3ffe::/16              1    12
~~~~~~~~~~

This preference table update moves 2002::/16 to de-preference status in line with RFC 7526 and changes the default address selection to move fc00::/7 above legacy IPv4, changing ::ffff:0:0/96 to preference 30. 

Rule 5.5 of RFC 6724 states:

~~~~~~~~~~
Rule 5.5: Prefer addresses in a prefix advertised by the next-hop.
If SA or SA's prefix is assigned by the selected next-hop that will
be used to send to D and SB or SB's prefix is assigned by a different
next-hop, then prefer SA.  Similarly, if SB or SB's prefix is
assigned by the next-hop that will be used to send to D and SA or
SA's prefix is assigned by a different next-hop, then prefer SB.
Discussion: An IPv6 implementation is not required to remember
which next-hops advertised which prefixes.  The conceptual models
of IPv6 hosts in Section 5 of [RFC4861] and Section 3 of [RFC4191]
have no such requirement.  Hence, Rule 5.5 is only applicable to
implementations that track this information.
~~~~~~~~~~

This document updates RFC 6724 section 5.5 to the following:

~~~~~~~~~~
Rule 5.5: Hosts MUST prefer addresses in a prefix advertised by the next-hop.
If SA or SA's prefix is assigned by the selected next-hop that will
be used to send to D and SB or SB's prefix is assigned by a different
next-hop, then prefer SA.  Similarly, if SB or SB's prefix is
assigned by the next-hop that will be used to send to D and SA or
SA's prefix is assigned by a different next-hop, then prefer SB.
Discussion: An IPv6 implementation is not required to remember
which next-hops advertised which prefixes.  The conceptual models
of IPv6 hosts in Section 5 of [RFC4861] and Section 3 of [RFC4191]
have no such requirement.  Hence, Rule 5.5 is only applicable to
implementations that track this information.
~~~~~~~~~~

# The practicalities of implementing address selection support

As with most adjustments to standards, and using RFC 6724
itself as a measuring stick, the updates defined in this document will likely take between 8-20 years to become common enough for consistent behavior within most operating systems. At the time of writing, it has been over 10 years since RFC 6724
has been published but we continue to see existing commercial and open source operating systems exhibiting {{RFC3484}}
behavior. 

While it should be noted that RFC 6724 defines a solution that is functional theoretically, operationally the solution of adjusting the address preference selection table
is both operating system dependent and unable to be signaled by any network mechanism such as within a router advertisement or DHCPv6 option (while {{RFC7078}} defines such a DHCPv6 option, it is not by any means widely implemented). This lack of an
intra-protocol or network-based ability to adjust address selection preference, along with the inability to adjust a notable number of operating systems either programmatically or manually
renders operational scalability of such a mechanism challenging.  

It is especially important to note this behavior in the long lifecycle equipment that exists in industrial control and operational technology environments due to their very long mean time to replacement/lifecycle.

In practice this means that network operators and those who design networks need to keep these considerations in mind.  One workaround should the ULA and IPv4 preference issue be of concern is to use IPv6-only networking, and to simply not deploy dual-stack. Another is to use GUA IPv6 addresses, which are preferred by defaul over all IPv4 addresses.

# Notes on the 6Man Working Group list discussion

Authors' note for the -00 version: this section captures some interesting suggestions from the 300 or so emails in the past few months in the 6man WG on this topic. These are noted, and captured here to inform discussion of the draft should it move forward in the WG.

* The suggestion to automatically insert an observed ULA /48 into the policy table to elevate a locally used ULA above IPv4 and GUA addresses was quite popular, though kernel implementation may be challenging for all platforms. This would be supported by changing the “MAY" in Section 2.1 and the “might” in Section 10.6 of RFC 6724 to “SHOULD” (or even a MUST). The case for a MUST is greater in order to allow for maximum network operator flexibility if the source selection table is not modified by the operating system. This could be an acceptable compromise, but requires two additional additions to an IPv6 ULA network: router manufacturers must now implement this new feature that is not a standard option in IPv6 Router Advertisements (RAs) and operators must know that the capability to add a tag for ULA prefixes in the source selection table is an operational possibility and now part of an architectural consideration. Network operators using managed addressing may have not considered using a tagged ULA prefix in RA as an option.

* The list discussed handling of corner cases, though what constitutes a corner case is in itself not wholly clear. The above suggestion for example would not cover the case where two sites using ULAs merged, and multiple ULA prefixes needed to be considered local. The open question is how deeply we consider corner cases; is some requirement for explicit configuration of certain cases inevitable? Is improving the current situation sufficient?

* A suggestion to use an RA PIO with A=0 and L=0, based on an interpretation of Section 2.1 of RFC 8028, was proposed but considered something of a stretch. That said, it could be an RA-based starting point to give some configurability for non-DHCPv6 networks.

# Acknowledgements 

The authors would like to acknowledge the valuable input and contributions of the 6man WG including Brian Carpenter, XiPeng Xiao, Eduard Vasilenko, David Farmer, Bob Hinden, Ed Horley, Tom Coffeen, Scott Hogg, and Chris Cummings. 

# Security Considerations

There are no direct security considerations in this document. 

The mixed preference for IPv6 over IPv4 from the default policy table in RFC 6724 represents a potential security issue, given an operator may expect ULAs to be used when in practice RFC 1918 addresses are used instead.

When using the updated ULA source address selection defined in this document, network operators MUST follow Section 4.3 of {{RFC4193}} for firewall/packet filtering as "routers be configured by default to keep any packets with Local
IPv6 addresses from leaking outside of the site and to keep any site prefixes from being advertised outside of their site." Following this security practice is critical when ULAs have more broad reachability.

# IANA Considerations

None.

# Appendix A. Changes since RFC6724

* Update to default preference table moving 6to4 address block 2002::/16 to de-preference status in line with {{RFC7526}} 
* Change the default address selection to move fc00::/7 to preference 35, above legacy IPv4,  
* Change ::ffff:0:0/96 to preference 30.
* Change section 5.5 Prefer addresses in a prefix advertised by the next-hop to a MUST

--- back
