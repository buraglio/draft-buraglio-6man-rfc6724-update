---
title: Updates to host address source selection
docname: draft-buraglio-6man-rfc6724-update
cat: std

author:
      -
        ins: N. Buraglio
        name: Nick Buraglio
        org: Energy Sciences Network
        email: buraglio@forwardingplane.net

author:
      -
        ins: T. Chown
        name: Tim Chown
        org: JISC
        email: Tim.Chown@jisc.ac.uk

author:
      -
        ins: B. Carpenter
        name: Brian Carpenter
        org: Univ. of Auckland
        email: brian.e.carpenter@gmail.com

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

--- note Lorem Ipsum


--- abstract

The behavior of ULA addressing as defined by [](RFC6724) is preferred below legacy IPv4 addressing, thus rendering ULA IPv6 deployment functionally unusable in IPv4 / IPv6 dual-stacked environments. This behavior is counter to the operational behavior of GUA IPv6 addressing on nearly all modern operating systems that leverage a preference model based on [](RFC6724).
This document outlines buth the operational implications of section section 10.6 of [](RFC6724) as described in [](draft-ietf-v6ops-ula) and updates the process to better suit operational deployment and management of Unique Local Addressing (ULA) in production.

--- middle

Introduction
============

In modern IPv4 / IPv6 dual-stacked environments, ULA addressing and GUA IPv6 addressing exhibit opposite behavior, which creates difficulties in deployments
leveraging ULA addressing. This conflicting behavior carries planning, operational, and security implications for environments requiring ULA addressing with IPv4/IPv6 dual-stack and prioritization of IPv6 traffic by default, as is the behavior with IPv6 GUA addressing.

-----------

# Defining Well Known Unintended Operational Issues With ULA

The [](RFC6724) definition is incomplete for ULA precedence if a host is operating in a dual-stack environment.
As written, [](RFC6724) section 10.3 states:

~~~~~~~~~~
"The default policy table gives IPv6 addresses higher precedence than
IPv4 addresses.  This means that applications will use IPv6 in
preference to IPv4 when the two are equally suitable.  An
administrator can change the policy table to prefer IPv4 addresses by
giving the ::ffff:0.0.0.0/96 prefix a higher precedence".
~~~~~~~~~~

Expected behavior would be that ULA address space would be preferred over legacy IPv4, however this is not the case. This presents an acute issue with any environment that will use ULA addressing long side legacy IPv4 that is counter to the standard expectations for legacy IPv4 / IPv6 dual-stack behavior of preferring IPv6, as is performed with GUA addressing.
Further, [](RFC6724) Section 10.6 states that this is resolvable by adding a site-specific policy to cause ULAs
within a site to be preferred over global addresses. While theoretically possible, this presents significant issues on devices with inaccessable configuration files as detailed below.

## Operational Implications

There are demonstrated and easily repeatable uses cases of ULA not being preferred in some OS and network equipment over legacy IPv4 that necessitate the immediate update to [](RFC6724)
to better reflect the original intent of the RFC. As with most adjustments to standards, and using [](RFC6724)
itself as a measurement, this update will likely take between 8-20 years to become common enough for relatively consistent behavior within operating systems. As a reference, as of the time of this writing, it has been 10 years since [](RFC6724)
has been published but we continue to see existing commercial and open source operating systems exhibiting [](RFC3484)
behavior. While it should be noted that [](RFC6724) defines a solution that is functional academically, operationally the solution of adjusting the address preference selection table
is both operating system dependent and unable to be signaled by any network mechanism such as within a router advertisement, DHCPv6 option, or the like. This lack of an
intra-protocol or network-based ability to adjust address selection preference, along with the inability to adjust a notable number of operating systems either programmatically or manually
renders operational scalability of such a mechanism functionally untenable.  
It is anticipated that any update of [](RFC6724) would require an additional 8-20 years to be fully realized and properly implemented in a majority of network connected systems. In addition, in the current versions of Linux,
the priority table (gai.conf) still makes reference to [](RFC3484), further demonstrating the long timeframe to have updates reflected in a current, modern operating system. Examples of such out-of-date behavior can be found in printers, cameras, fixed devices, IoT sensors, and longer lifecycle equipment.
It is especially important to note this behavior in the long lifecycle equipment that exists in industrial control and operational technology environments due to their very long mean time to replacement.
The core issue is the stated interpretation from gai.conf that has the following default:

~~~~~~~~~~
#scopev4  <mask> <value> 
#  Add another rule to the RFC 6724 scope table for IPv4 addresses. 
#    By default the scope IDs described in section 3.2 in RFC 6724 are
#    used.  Changing these defaults should hardly ever be necessary.
#    The defaults are equivalent to:
#
#scopev4 ::ffff:169.254.0.0/112  2
#scopev4 ::ffff:127.0.0.0/104    2
#scopev4 ::ffff:0.0.0.0/96       14
~~~~~~~~~~

{: #scope4 title="gai.conf example" alt="gai.conf" }

Notice that they are interpreting the legacy IPv4 address range as "scopev4" and the prefix ::ffff:0.0.0.0/96 which has a higher precedence (35) in [](RFC6724) then the ULA prefix of fc00::/7 (3). This results in legacy IPv4 being preferred over IPv6 ULA. While not inherently undesirable, the operational outcome when utilizing dual-stack with ULA is inconsistent and imparts unnecessary difficulty for both troubleshooting and creating the requisite baseline of expected behavior which are both requirements for supportable production deployments. This results in operational and engineering teams not gaining IPv6 experience as limited traffic is actually using IPv6, and security baseline expectations can, depending on the host implementation, be inconsistent at best and haphazard at worst.

In practice, [](RFC6724) imposes several operational shortcomings preventing both consistent and desired behavior. If we define "desired behavior" as IPv6 preference over legacy IPv4 for address and protocol selection, then the resulting implemented behavior, based on [](RFC6724), will fall short of that intent. Based on the current verbiage, dual-stacked hosts configured with both a legacy IPv4 address and an IPv6 ULA address, the resulting behavior will manifest as a host choosing IPv4 over ULA IPv6. This behavior deviates from the current goal of a host with legacy IPv4 address and also with an IPv6 GUA address preferring IPv6 over IPv4. Operationally and strategically, this manifests as an impediment to deployment of IPv6 for many non service provider and mobile networks phasing in dual-stacked (both legacy IPv4 and IPv6) networking with the expectation of consistent behavior (i.e. always use IPv6 before legacy IPv4).

Other operational considerations are the use of the policy table detailed in section 2.1 of [](RFC6724). While conceptually the intent was for a configurable, longest-match table to be adjusted as-needed. In practice, modifying the prefix policy table remains difficult across platforms, and in some cases impossible. Embedded, proprietary, closed source, and IoT devices are especially difficult to adjust, and are in many cases incapable of any adjustment whatsoever. Large scale manipulation of the policy table also remains out of the realm of realistic support for small and medium scale operators due to lack of ability to manipulate all the hosts and systems, or a lack of tooling and access.

Below is an example of a gai.conf file from a modern Linux installation as of 03 April 2022:

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

Several assumptions are made here and are largely based on interpretations of [](RFC6724)
but are not operationally relevant in modern networks. As this file or an equivalent structure within a given operating system is referenced, it dictates the
behavior of the getaddrinfo() or analogous process. More specifically, where getaddrinfo() or comparable API is used, the sorting behavior should take into account both
the source address of the requesting host as well as the destination addresses returned and sort according to both source and destination addressing, i.e, when a ULA address is
returned, the source address selection should return and use a ULA address if available. Similarly, if a GUA address is returned the source address selection should return
a GUA source address if available.

Here are some example failure modes:

* ULA per [](RFC6724) is less preferred (the Precedence value is lower) than all legacy IPv4 (represented by ::ffff:0:0/96 in the aforementioned table).
* Because of the lower Precedence value of fc00::/7, if a host has legacy IPv4 enabled, it will use legacy IPv4 before using ULA.
* A dual-stacked client will source the traffic from the legacy IPv4 address, meaning it will require a corresponding legacy IPv4 destination address.

Per number 3, even a host choosing a destination with A and AAAA DNS records, the host in question will choose the A record to get an legacy IPv4 address for the destination, meaning ULA IPv6 is rendered completely unused. It is also notable that Happy Eyeballs [](RFC8305) will not change the source address selection process on a host. Happy Eyeballs will only modify the destination sorting process.

As a direct result of the described failure modes, and in addition to the aforementioned operational implications, use of ULA is not a viable option for dual-stack networking transition planning, large scale network modeling, network lab environments or other modes of emulating a large scale networking that runs both IPv4 and IPv6 concurrently with the expectation that IPv6 will be preferred by default.

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

# Smoldering ideas

fixes without a global precedence for ULAs:
set PIO with L=0 and A=0 (exactly as recommended in RFC 8028, but for other reasons). If a host sees such a PIO for a ULA prefix, it could serve as a signal that the prefix is to be given a suitable precedence, even though it is not on-link and not used for SLAAC.

Insert the /48 for any directly connected /64, utilize some RIO/PIO L=0/A=0 extension that adds additional non-direct /48.


# References {#refstyle}

The IETF documents referred to here are [](RFC2119) and [](I-D.ietf-core-coap).  We also refer to [REST](REST).  [](a-rose) and [](exampleTable) are explicitly tagged, as is [](refstyle), but [](security-considerations) is implicit.  The web page at [ipv.sx](http://ipv.sx/) is not part of this document.

# Security Considerations

None.

# IANA Considerations

None.

# Appendix A. Acknowledgements 
The authors would like to acknowledge the valuable input and contributions of David Farmer, Bob Hinton, Ed Horley, Tom Coffeen, Scott Hogg, and Chris Cummings. 

# Appendix B. Changes since RFC6724

*Update to default preference table moving 6to4 address block 2002::/16 to de-preference status in line with [](RFC7526) 
*Change the default address selection to move fc00::/7 to preference 35, above legacy IPv4,  
*Change ::ffff:0:0/96 to preference 30. \
*Change section 5.5 Prefer addresses in a prefix advertised by the next-hop to a MUST

--- back