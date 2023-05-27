---
title: Preference for ULAs over RFC1918 addresses in RFC6724
docname: draft-buraglio-6man-rfc6724-update
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
  I-D.ietf-core-coap:

informative:
  REST:
    title:
    author:
        ins:
        name:
        org:
    date:

--- abstract

This document updates [](RFC6724) based on operational experience gained since its publication over ten years ago. In particular it updates the preference of Unique Local Addresses (ULAs) in the default address selection policy table, which as originally defined by [](RFC6724) has lower precendence than legacy IPv4 addressing. The update places both IPv6 Global Unicast Addresses (GUAs) and ULAs ahead of all IPv4 addresses on the policy table to better suit operational deployment and management of ULAs in production. This document also updates requirements on configurability of the policy table and preference for using ddresses from a prefix advertised by a next-hop router, and demotes the preference for 6to4 addresses in the default policy table.

--- middle

# Introduction
============

When [](RFC6724) was published in 2012 it was expected that the default policy table may need to be updated from operartional experience; section 2.1 says "It is important that implementations provide a way to change the default policies as more experience is gained" and points to the expamples in Section 10, including Section 10.6 which considers a ULA example.

This document is written on the basis of operational experience, in particular for scanerios where ULAs are used within a site. The current default policy table in RFC6724 leads to preference for IPv6 GUAs over IPv4 globals, which is widely considered to be preferential behaviour to support greater use of IPv6 in dual-stack environments, and to allow sites to phase out IPv4 as its use becomes ever lower.

However, the  default policy table also puts IPv6 ULAs below all IPv4 addresses, including RFC 1918 addresses. For many site operators this behavior will be counter intuitive, and may create difficulties with respect to planning, operational, and security implications for environments where ULA addressing is used in certain IPv4/IPv6 dual-stack network scenarios. The expected prioritization of IPv6 traffic over IPv4 by default, as happens IPv6 GUA addressing, will not happen for ULAs.

An IPv6 deployment, whether enterprise, residential or other, may use IPv6 GUAs, IPv6 ULAs, IPv4 globals, IPv4 RFC1918 addressing, and may or may not use some form of NAT. This document makes no comment or recommendation on how ULAs are used, or on NAT, but notes that operationally where GUAs and ULAs are used alongside RFC1918 addressing, an IPv6 GUA would be selected to reach an IPv6 GUA destination, but where only ULAs and RFC1918 addressing are used, RFC1918 addresses will be preferred.

This document updates the default policy table to elevate the preference for ULAs such that ULAs will be preferred over all IPv4 addresses, providing more consistent and less confusing behaviour for operators.

This issue also highlights the need for the original RFC6724 address slection policy table to be configurable. RFC6724 Section 2.1 states that the table SHOULD be configurable; this document proposes elevating that requirement to MUST, to ensure that any device can have its polict bale tuned for the scenario in which it is deployed. Section 10 of RFC6724 gives other examples of why configurability is important.

It has also become clearer from operational experience that the heuristic to prefer addresses drawn from a prefix advertiused by a next-hop router is a valuable one to use.  This text therefore also proposes elevating that requirement in Section 5.5 from SHOULD to MUST.

These updates are discussed in more detail in the following sections, with a further section providing a summary of the updates.

-----------

# Unintended Operational Issues Regarding IPv6 Preference and ULAs

The preference for use of IPv6 addressing over IPv4 addressing in [](RFC6724) is inconsistent. As written, [](RFC6724) section 10.3 states:

~~~~~~~~~~
"The default policy table gives IPv6 addresses higher precedence than
IPv4 addresses.  This means that applications will use IPv6 in
preference to IPv4 when the two are equally suitable.  An
administrator can change the policy table to prefer IPv4 addresses by
giving the ::ffff:0.0.0.0/96 prefix a higher precedence".
~~~~~~~~~~

The expected behavior would be that ULA address space would be preferred over legacy IPv4, however this is not the case. This presents an issue with any environment that will use ULA addressing alongside legacy IPv4, whether global or RFC1918. This is counter to the standard expectations for legacy IPv4 / IPv6 dual-stack behavior in preferring IPv6, which is the case for GUA addressing.

## Operational Implications

There are demonstrated and easily repeatable uses cases of ULA not being preferred in some OS and network equipment over legacy IPv4 that necessitate an update to [](RFC6724)
to better reflect the original intent of the RFC. 

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

The legacy IPv4 address range in the gai.conf file is "scopev4" and the prefix ::ffff:0.0.0.0/96 which has a higher precedence (35) in [](RFC6724) than the ULA prefix of fc00::/7 (3). This results in legacy IPv4 being preferred over IPv6 ULA. While not inherently undesirable, the operational outcome when utilizing dual-stack with ULA is inconsistent and imparts unnecessary difficulty for both troubleshooting and creating the requisite baseline of the expected behavior which are both requirements for supportable production deployments. Depending on the host implementation, security baseline expectations can be inconsistent at best and haphazard at worst.

As the gai.conf file, or an equivalent within a given operating system, is referenced it dictates the
behavior of the getaddrinfo() or analogous process. More specifically, where getaddrinfo() or a comparable API is used, the sorting behavior should take into account both
the source address of the requesting host as well as the destination addresses returned and sort according to both source and destination addresses, i.e, when a ULA address is
returned, the source address selection should return and use a ULA address if available. Similarly, if a GUA address is returned the source address selection should return a GUA source address if available.

However, there are clearly evidenced example of failure scenarios:

1. ULA per [](RFC6724) is less preferred (the Precedence value is lower) than all legacy IPv4 (represented by ::ffff:0:0/96 in the aforementioned table).
2. Because of the lower Precedence value of fc00::/7, if a host has legacy IPv4 enabled, it will use legacy IPv4 before using ULA.
3. A dual-stacked client will source the traffic from the legacy IPv4 address, meaning it will require a corresponding legacy IPv4 destination address.

For scenario number 3, when a host resolves through DNS a destination with A and AAAA DNS records, the host will choose the A record to get an legacy IPv4 address for the destination, meaning ULA IPv6 is rendered unused. 

As a result, the use of ULAs is not a viable option for dual-stack networking transition planning, large scale network modeling, network lab environments or other modes of large scale networking that run both IPv4 and IPv6 concurrently with the expectation that IPv6 will be preferred by default.

# Configurability of the Default Policy Table

In principle the above problem would not be an issue were the [](RFC6724) default policy table readily configurable in all systems. Section 10.6 states that ULAs can be preferred by adding a site-specific entry to the default policy table. In practice, this is currently not always possible.

While conceptually the intent was for a configurable, longest-match table to be adjusted as needed. In practice, modifying the prefix policy table remains difficult across platforms, and in some cases impossible. Embedded, proprietary, closed source, and IoT devices are especially difficult to adjust, and are in many cases incapable of any adjustment. Large scale manipulation of the policy table also remains out of the realm of realistic support for small and medium scale operators due to lack of ability to manipulate all the hosts and systems, or a lack of tooling and access.

Operational experience suggests that the default policy table needs to be as configurable as possible in as many systems as possible. This update therefore proposes that the requirement that IPv6 implementations support configurable address selection
via a mechanism at least as powerful as the policy table be elevated from a SHOULD to a MUST.

Authors' note for the -00 version: of course we state above that for some platforms, the ability to implement such a method is challenging.  The question for the 6man WG is how to ensure configurability is as widespread as possible.

# Next-hop router heuristic

The heuristic for address selection defined in Section 5.5 of RFC6724 to prefer addresses in a prefix advertised by a next-hop router has proven to be very useful.  RFC6724 does not state any requirement for SHOULD or MUST for this heuristic to be used; this update therefore proposes stating that the application of the heuristic be a MUST.

# Preference of 6to4 addresses

The anycast prefix for 6to4 relays was deprecated by [](RFC7526) in 2015, and since that time the use of 6to4 addressing has further declined to the point where it is generally not seen and can be considered to all intents and purposes deprecated in use.  This document therefore demotes the preference of the 6to4 prefix in the policy table to the same minimum preference as carried by the deprecated site local and 6bone address prefixes.

# Adjustments to RFC6724

Rule 2.1 of [](RFC6724) states:

~~~~~~~~~~
IPv6 implementations SHOULD support configurable address selection
via a mechanism at least as powerful as the policy tables defined
here.  It is important that implementations provide a way to change
the default policies as more experience is gained.  Sections 10.3
through 10.7 provide examples of the kind of changes that might be
needed.
~~~~~~~~~~

This document updates [](RFC6724) section 2.1 to the following:

~~~~~~~~~~
IPv6 implementations MUST support configurable address selection
via a mechanism at least as powerful as the policy tables defined
here.  It is important that implementations provide a way to change
the default policies to ensure operational supportability and flexibility in deployment.
Sections 10.3 through 10.7 provide examples of the kind of changes that might be
needed.
~~~~~~~~~~

Rule 2.1 of [](RFC6724) further states:

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

This document updates [](RFC6724) section 2.1 to the following:

## needs labels adjusted 
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

This preference table update moves 2002::/16 to de-preference status in line with [](RFC7526) and changes the default address selection to move fc00::/7 above legacy IPv4, changing ::ffff:0:0/96 to preference 30. 

Rule 5.5 of [](RFC6724) states:

~~~~~~~~~~
Rule 5.5: Prefer addresses in a prefix advertised by the next-hop.
If SA or SA's prefix is assigned by the selected next-hop that will
be used to send to D and SB or SB's prefix is assigned by a different
next-hop, then prefer SA.  Similarly, if SB or SB's prefix is
assigned by the next-hop that will be used to send to D and SA or
SA's prefix is assigned by a different next-hop, then prefer SB.
Discussion: An IPv6 implementation is not required to remember
which next-hops advertised which prefixes.  The conceptual models
of IPv6 hosts in Section 5 of [](RFC4861) and Section 3 of [](RFC4191)
have no such requirement.  Hence, Rule 5.5 is only applicable to
implementations that track this information.
~~~~~~~~~~

This document updates [](RFC6724) section 5.5 to the following:

## needs eyes

~~~~~~~~~~
Rule 5.5: Hosts MUST prefer addresses in a prefix advertised by the next-hop.
If SA or SA's prefix is assigned by the selected next-hop that will
be used to send to D and SB or SB's prefix is assigned by a different
next-hop, then prefer SA.  Similarly, if SB or SB's prefix is
assigned by the next-hop that will be used to send to D and SA or
SA's prefix is assigned by a different next-hop, then prefer SB.
Discussion: An IPv6 implementation is not required to remember
which next-hops advertised which prefixes.  The conceptual models
of IPv6 hosts in Section 5 of [](RFC4861) and Section 3 of [](RFC4191)
have no such requirement.  Hence, Rule 5.5 is only applicable to
implementations that track this information.
~~~~~~~~~~

# The practicalities of implementing address selection support

As with most adjustments to standards, and using [](RFC6724)
itself as a measurement, the updates defined in this document will likely take between 8-20 years to become common enough for consistent behavior within most operating systems. At the time of writing, it has been over 10 years since [](RFC6724)
has been published but we continue to see existing commercial and open source operating systems exhibiting [](RFC3484)
behavior. 

While it should be noted that [](RFC6724) defines a solution that is functional theoretically, operationally the solution of adjusting the address preference selection table
is both operating system dependent and unable to be signaled by any network mechanism such as within a router advertisement or DHCPv6 option (while RFC7078 defines such a DHCPv6 option, it is not by any means widely implemented). This lack of an
intra-protocol or network-based ability to adjust address selection preference, along with the inability to adjust a notable number of operating systems either programmatically or manually
renders operational scalability of such a mechanism challenging.  

It is especially important to note this behavior in the long lifecycle equipment that exists in industrial control and operational technology environments due to their very long mean time to replacement/lifecycle.

In practice this means that network operators and those who design networks need to keep these considerations in mind.  One workaround should the ULA and IPv4 preference issue be of concern is to use IPv6-only networking, and to simply not deploy dual-stack. Another is to use GUA IPv6 addresses, which are preferred by defaul over all IPv4 addresses.

# Acknowledgements 

The authors would like to acknowledge the valuable input and contributions of Brian Carpenter, XiPeng Xiao, Eduard Vasilenko, David Farmer, Bob Hinton, Ed Horley, Tom Coffeen, Scott Hogg, and Chris Cummings. 

# References {#refstyle}

The IETF documents referred to here are [](RFC2119) and [](I-D.ietf-core-coap).  We also refer to [REST](REST).  [](a-rose) and [](exampleTable) are explicitly tagged, as is [](refstyle), but [](security-considerations) is implicit.  The web page at [ipv.sx](http://ipv.sx/) is not part of this document.

# Security Considerations

There are no direct security considerations in this document. 

The mixed preference for IPv6 over IPv4 from the default policy table in RFC6724 represents a potential security issue, given an operator may expect ULAs to be used when in practice RFC1918 addresses are used instead.

When using the updated ULA source address selection defined in this document, network operators must follow Section 4.3 of [](RFC4193) for firewall/packet filtering as "routers be configured by default to keep any packets with Local
IPv6 addresses from leaking outside of the site and to keep any site prefixes from being advertised outside of their site." Following this security practice is critical when ULAs have more broad reachability.

# IANA Considerations

None.

# Appendix A. Changes since RFC6724

* Update to default preference table moving 6to4 address block 2002::/16 to de-preference status in line with [](RFC7526) 
* Change the default address selection to move fc00::/7 to preference 35, above legacy IPv4,  
* Change ::ffff:0:0/96 to preference 30. \
* Change section 5.5 Prefer addresses in a prefix advertised by the next-hop to a MUST

--- back
