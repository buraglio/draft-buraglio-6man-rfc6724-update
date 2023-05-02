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

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur nibh mi, mollis varius imperdiet id, venenatis ut nisi. Phasellus mauris urna, ultrices at massa id, faucibus malesuada nisi.

### Lorem ipsum

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur nibh mi, mollis varius imperdiet id, venenatis ut nisi. Phasellus mauris urna, ultrices at massa id, faucibus malesuada nisi.

### Blockquote

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur nibh mi, mollis varius imperdiet id, venenatis ut nisi. Phasellus mauris urna, ultrices at massa id, faucibus malesuada nisi.

> This is a blockquoted paragraph.  And some more.
> And some more.  And some more.  And some more.  And some more.
> And some more.  And some more.  And some more.  And some more.

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur nibh mi, mollis varius imperdiet id, venenatis ut nisi. Phasellus mauris urna, ultrices at massa id, faucibus malesuada nisi.

### Section with a very long title that will overflow the bounds of both the rendering and the table of contents

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



