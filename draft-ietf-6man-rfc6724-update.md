---
title: Preference for IPv6 ULAs over IPv4 addresses in RFC6724
abbrev: Prefer ULAs over IPv4 addresses
docname: draft-ietf-6man-rfc6724-update-00
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
 
  

--- abstract

This document updates RFC 6724 based on operational experience gained since its publication over ten years ago. In particular it updates the precedence of Unique Local Addresses (ULAs) in the default address selection policy table, which as originally defined by RFC 6724 has lower precedence than legacy IPv4 addressing. The update places both IPv6 Global Unicast Addresses (GUAs) and ULAs ahead of all IPv4 addresses on the policy table to better suit operational deployment and management of ULAs in production. In updating the RFC 6724 default policy table, this document also demotes the preference for 6to4 addresses. These changes to default behavior improve supportability of common use cases such as, but not limited to, automatic / unmanaged scenarios. It is recognized that some less common deployment scenarios may require explicit configuration or custom changes to achieve desired operational parameters.  

--- middle

# Introduction

When {{RFC6724}} was published in 2012 it was expected that the default policy table may need to be updated from operational experience; section 2.1 says "It is important that implementations provide a way to change the default policies as more experience is gained" and points to the examples in Section 10, including Section 10.6 which considers a ULA example.

This document is written on the basis of such operational experience, in particular for scenarios where ULAs are used within a site. 

The current default policy table in RFC 6724 leads to preference for IPv6 GUAs over IPv4 globals, which is widely considered to be preferential behavior to support greater use of IPv6 in dual-stack environments, and to allow sites to phase out IPv4 as its use becomes ever lower.

However, the default policy table also puts IPv6 ULAs below all IPv4 addresses, including {{RFC1918}} addresses. For many site operators this behavior will be counter-intuitive, and may create difficulties with respect to planning, operational, and security implications for environments where ULA addressing is used in certain IPv4/IPv6 dual-stack network scenarios. The expected prioritization of IPv6 traffic over IPv4 by default, as happens with IPv6 GUA addressing, will not happen for ULAs.

An IPv6 deployment, whether enterprise, residential or other, may use combinations of IPv6 GUAs, IPv6 ULAs, IPv4 globals, IPv4 RFC 1918 addressing, and may or may not use some form of NAT. 

This document makes no comment or recommendation on how ULAs are used, or on the use of NAT in an IPv6 network. As the default policy table stands, operationally where GUAs and ULAs are used alongside RFC 1918 addressing, an IPv6 GUA would be selected to reach an IPv6 GUA destination.  However where there are only ULAs and RFC1918 addressing in use, RFC 1918 addresses will be preferred.

This document updates the default policy table to elevate the preference for ULAs such that ULAs will be preferred over all IPv4 addresses, providing more consistent and less confusing behavior for operators. 

This change aims to improve the default handling of address selection for common cases, and unmanaged / automatic scenarios rather than those where DHCPv6 is deployed. Sites using DHCPv6 for host configuration management can make use of implementations of {{RFC7078}} to apply changes to the RFC 6724 policy table.

The changes should also assist operators in phasing out IPv4 from dual-stack environments, since IPv6 GUAs and ULAs will be preferred over any IPv4 addresses.  This is an extremely important enabler towards IPv6-only networking.

The changes are discussed in more detail in the following sections, with a further section providing a summary of the proposed updates.

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

There are demonstrated and easily repeatable operational examples of the impact of the current RFC 6724 behaviour, i.e., ULAs not being preferred in OS and network equipment over legacy IPv4 addresses. These reinforce the need to update RFC 6724 to both better reflect the original intent of the RFC and to facilitate the depreciation and eventual removal of IPv4 in network environments. 

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

The legacy IPv4 address range in the gai.conf file is "scopev4" and the prefix ::ffff:0.0.0.0/96 which has a higher precedence (35) in RFC 6724 than the ULA prefix of fc00::/7 (3). This results in legacy IPv4 being preferred over IPv6 ULA. The result is that the operational outcome when utilizing dual-stack with ULA is inconsistent and imparts unnecessary difficulty for both troubleshooting and creating the requisite baseline of the expected behavior, which are both requirements for supportable production deployments. Depending on the host implementation, security baseline expectations can be inconsistent at best and haphazard at worst.

As the gai.conf file, or an equivalent within a given operating system, is referenced it dictates the
behavior of the getaddrinfo() or analogous process. More specifically, where getaddrinfo() or a comparable API is used, the sorting behavior should take into account both
the source address of the requesting host as well as the destination addresses returned and sort according to both source and destination addresses, i.e, when a ULA address is
returned, the source address selection should return and use a ULA address if available. Similarly, if a GUA address is returned the source address selection should return a GUA source address if available.

However, there are clearly evidenced example of three failure scenarios:

1. ULA per RFC 6724 is less preferred (the precedence value is lower) than all legacy IPv4 (represented by ::ffff:0:0/96 in the aforementioned table).
2. Because of the lower precedence value of fc00::/7, if a host has legacy IPv4 enabled, it will use legacy IPv4 before using ULA.
3. A dual-stacked client will source the traffic from the legacy IPv4 address, meaning it will require a corresponding legacy IPv4 destination address.

For scenario number 3, when a host resolves through DNS a destination with A and AAAA DNS records, the host will choose the A record to get an legacy IPv4 address for the destination, meaning ULA IPv6 is rendered unused. 

As a result, the use of ULAs is not a viable option for dual-stack networking transition planning, large scale network modeling, network lab environments or other modes of large scale networking that run both IPv4 and IPv6 concurrently with the expectation that IPv6 will be preferred by default.

# Preference of 6to4 addresses

The anycast prefix for 6to4 relays was deprecated by {{RFC7526}} in 2015, and since that time the use of 6to4 addressing has further declined to the point where it is generally not seen and can be considered to all intents and purposes deprecated in use.  

This document therefore demotes the precedence of the 6to4 prefix in the policy table to the same minimum preference as carried by the deprecated site local and 6bone address prefixes.

# Adjustments to RFC 6724

Rule 2.1 of RFC 6724 states:

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
      fc00::/7              30    13
      ::ffff:0:0/96         20     4
      2001::/32              5     5
      2002::/16              5     2
      ::/96                  1     3
      fec0::/10              1    11
      3ffe::/16              1    12
~~~~~~~~~~

This preference table update moves 2002::/16 to de-preference its status in line with RFC 7526 and changes the default address selection to move fc00::/7 above legacy IPv4, with ::ffff:0:0/96 now set to precedence 20. 


# The practicalities of implementing address selection support

As with most adjustments to standards, and using RFC 6724 itself as a measuring stick, the updates defined in this document will likely take between 8-20 years to become common enough for consistent behavior within most operating systems. At the time of writing, it has been over 10 years since RFC 6724 has been published but we continue to see existing commercial and open source operating systems exhibiting {{RFC3484}} behavior. 

While it should be noted that RFC 6724 defines a solution to adjust the address preference selection table that is functional theoretically, operationally the solution is operating system dependent and cannot be signaled by any currently deployed network mechanism. While {{RFC7078}} defines such a DHCPv6 option, it is not by any means widely implemented. This lack of an intra-protocol or network-based ability to adjust address selection preference, along with the inability to adjust a notable number of operating systems either programmatically or manually renders operational scalability of such a mechanism challenging.  

It is especially important to note this behavior in the long lifecycle equipment that exists in industrial control and operational technology environments due to their very long mean time to replacement/lifecycle.

In practice this means that network operators and those who design networks need to keep these considerations in mind.  Should the current ULA and IPv4 preference issue be of concern then 'workarounds' do exist. One is to use IPv6-only networking, i.e., not deploy dual-stack, and another is to only use GUA IPv6 addresses which are preferred by default over all IPv4 addresses. 

# Notable changes in practice today

## Relation to {{RFC5220}}

The concerns expressed in section 2.2.2 of {{RFC 5220}} need to be considered. But with a separate label for ULA now present in the policy table, Rule 5 of Section 6 of RFC 6724 which states 
~~~~~
Rule 5: Prefer matching label.
   If Label(Source(DA)) = Label(DA) and Label(Source(DB)) <> Label(DB),
   then prefer DA.  Similarly, if Label(Source(DA)) <> Label(DA) and
   Label(Source(DB)) = Label(DB), then prefer DB.
~~~~~
means that the presence of the label and the rule defining the behavior based on said rule should prevent the situation described in that section from occurring.

## Relation to Happy Eyeballs

In cases where a ULA Source Address is selected to communicate with a GUA destination, Happy Eyeballs version 1 or 2 would result in practice in IPv4 being used since the ULA source address will likely fail (due to egress filtering at the border, as discussed in the Security Considerations below). Corner cases may exist where the ULA address will not fail, however, in normal operation these cases are more likely misconfigurations than intentional.

## Relation to {{RFC4193}}

If the operational guidelines in sections 4.1 and 4.3 of {{RFC4193}} are followed, a Destination Unreachable ICMPv6 Error should be received by the source device which should trigger Happy Eyeballs. If Happy Eyeballs is not implemented, the next option should follow the guidelines outlined in {{RFC4193}}.

# Acknowledgements 

The authors would like to acknowledge the valuable input and contributions of the 6man WG including Brian Carpenter, XiPeng Xiao, Eduard Vasilenko, David Farmer, Bob Hinden, Ed Horley, Tom Coffeen, Scott Hogg, Chris Cummings, Paul Wefel, and Dale Carder. 

# Security Considerations

There are no direct security considerations in this document. 

The mixed preference for IPv6 over IPv4 from the default policy table in RFC 6724 represents a potential security issue, given an operator may expect ULAs to be used when in practice RFC 1918 addresses are used instead.

When using the updated ULA source address selection defined in this document, network operators MUST follow Section 4.3 of {{RFC4193}} for firewall/packet filtering as "routers be configured by default to keep any packets with Local
IPv6 addresses from leaking outside of the site and to keep any site prefixes from being advertised outside of their site." Following this security practice is critical when ULAs have more broad reachability.

Operators should be mindful of cases where one node is compliant with RFC 6724 as originally published and another node is compliant with the update presented in this document, as there may be inconsistent behaviour for communications initiated in each direction. Similarly, differences between current RFC 6724-compliant and RFC 3484-compliant nodes may also be observed. Ultimately all nodes should be made compliant to the updated specification described in this document.

# IANA Considerations

None.

# Appendix A. Changes since RFC6724

* Update to default preference table moving 6to4 address block 2002::/16 to de-preference status in line with {{RFC7526}} 
* Change the default address selection to move fc00::/7 to preference 30, above legacy IPv4,  
* Change ::ffff:0:0/96 to preference 20.

--- back
