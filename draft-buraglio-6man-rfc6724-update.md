---
title: Updates to host address source selection 
docname: draft-buraglio-6man-rfc6724-update
cat: std

author:
      -
        ins: N. Buraglio
        name: Nick Buraglio
        org: Energy Sciences Network
        abbrev: 
        street:
          - 
          - 
        country: USA
        email: buraglio@forwardingplane.net

author:
      -
        ins: T. Chown
        name: Tim Chown
        org: JISC
        abbrev: 
        street:
          - 
          - 
        country: UK
        email: Tim.Chown@jisc.ac.uk

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

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur nibh mi, mollis varius imperdiet id, venenatis ut nisi. Phasellus mauris urna, ultrices at massa id, faucibus malesuada nisi.

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur nibh mi, mollis varius imperdiet id, venenatis ut nisi. Phasellus mauris urna, ultrices at massa id, faucibus malesuada nisi.


--- abstract

The behavior of ULA addressing as defined by [](RFC6724) is preferred below legacy IPv4 addressing, thus rendering ULA IPv6 deployment functionally unusable in IPv4 / IPv6 dual-stacked environments. This behavior is counter to the operational behavior of GUA IPv6 addressing on nearly all modern operating systems that leverage a preference model based on [](RFC6724).
This document outlines buth the operational implications of section section 10.6 of [](RFC6724) as described in [](draft-ietf-v6ops-ula) and updates the process to better suit operational deployment and management of Unique Local Addressing (ULA) in production.


--- middle

Introduction
============

In modern IPv4 / IPv6 dual-stacked environments, ULA addressing and GUA IPv6 addressing exhibit opposite behavior, which creates difficulties in deployments
leveraging ULA addressing. This conflicting behavior carries planning, operational, and security implications for environments requiring ULA addressing with IPv4/IPv6 dual-stack and prioritization of IPv6 traffic by default, as is the behavior with IPv6 GUA addressing.

-----------


### Defining Well Known Unintended Operational Issues With ULA

The [](RFC6724) definition is incomplete for ULA precedence if a host is operating in a dual-stack environment. 
    As written, [](RFC6724) section 10.3 states: 

>"The default policy table gives IPv6 addresses higher precedence than
> IPv4 addresses.  This means that applications will use IPv6 in
> preference to IPv4 when the two are equally suitable.  An
> administrator can change the policy table to prefer IPv4 addresses by
> giving the ::ffff:0.0.0.0/96 prefix a higher precedence".

Expected behavior would be that ULA address space would be preferred over legacy IPv4, however this is not the case. This presents an acute issue with any environment that will use ULA addressing long side legacy IPv4 that is counter to the standard expectations for legacy IPv4 / IPv6 dual-stack behavior of preferring IPv6, as is performed with GUA addressing. 
Further, [](RFC6724) Section 10.6 states that this is resolvable by adding a site-specific policy to cause ULAs
within a site to be preferred over global addresses. While theoretically possible, this presents significant issues on devices with inaccessable configuration files as detailed below.

### Operational Implications

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
# Lists

* Item1
* Item2
  * Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur nibh mi, mollis varius imperdiet id, venenatis ut nisi. Phasellus mauris urna, ultrices at massa id, faucibus malesuada nisi.
  * ItemB
* Item3

This is a different block to separate the lists.

1. Item1
1. Item2
  1. ItemA
  1. ItemB
1. Item3
# Figures

~~~~~~~~~~
       _,--._.-,           
      /\_r-,\_ )           
   .-.) _;='_/ (.;         
    \ \'     \/S )         
     L.'-. _.'|-'          
    <_`-'\'_.'/            
      `'-._( \             
             \\       ___  
              \\   .-'_. / 
               \\ /.-'_.'  
                \('--'	   
                  \        
~~~~~~~~~~
{: #rose title="A Rose" alt="rose" }


# Tables

| Left-aligned | Center-aligned | Right-aligned|
| :-- | :--:|----:|
| Alpha | Bravo | Charlie |
| delta | echo | foxtrot |
| --- | --- | --- |
| xylophone | yankee | zulu |
{: #exampleTable title="Example Table"}

# References {#refstyle}

The IETF documents referred to here are [](RFC2119) and [](I-D.ietf-core-coap).  We also refer to [REST](REST).  [](a-rose) and [](exampleTable) are explicitly tagged, as is [](refstyle), but [](security-considerations) is implicit.  The web page at [ipv.sx](http://ipv.sx/) is not part of this document.

# Security Considerations

No, really.

# IANA Considerations

None


--- back

# Lorem ipsum

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur nibh mi, mollis varius imperdiet id, venenatis ut nisi. Phasellus mauris urna, ultrices at massa id, faucibus malesuada nisi.



