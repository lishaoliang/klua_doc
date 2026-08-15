> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: `portfs/doc/std_fat/` offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

# Front Matter / Legal

Hardware White Paper
Designing Hardware for Microsoft
 Operating Systems
Microsoft Extensible Firmware Initiative FAT32 File System Specification
FAT: General Overview of On-Disk Format

Version 1.03, December 6, 2000
Microsoft Corporation

The FAT (File Allocation Table) file system has its origins in the late 1970s and early1980s and was the file system supported by the Microsoft
 operating system. It was originally developed as a simple file system suitable for floppy disk drives less than 500K in size. Over time it has been enhanced to support larger and larger media. Currently there are three FAT file system types: FAT12, FAT16 and FAT32. The basic difference in these FAT sub types, and the reason for the names, is the size, in bits, of the entries in the actual FAT structure on the disk. There are 12 bits in a FAT12 FAT entry, 16 bits in a FAT16 FAT entry and 32 bits in a FAT32 FAT entry.

Contents
Notational Conventions in this Document
General Comments (Applicable to FAT File System All Types)
Boot Sector and BPB
FAT Data Structure
FAT Type Determination
FAT Volume Initialization
FAT32 FSInfo Sector Structure and Backup Boot Sector
FAT Directory Structure
FAT Long Directory Entries
Name Limits and Character Sets
Name Matching In Short & Long Names
Naming Conventions and Long Names
Effect of Long Directory Entries on Down Level Versions of FAT
Validating The Contents of a Directory
Other Notes Relating to FAT Directories

Microsoft, MS_DOS, Windows, and Windows
NT are trademarks or registered trademarks of Microsoft Corporation in the United States and/or other countries. Other product and company names mentioned herein may be the trademarks of their respective owners.
Microsoft
Corporation. All rights reserved.

Document History
Date
March 30, 2011
Updated the Legal Agreement
December 6, 2000
First publication
Microsoft Extensible Firmware Initiative FAT32 File System Specification
IMPORTANT-READ CAREFULLY: This Microsoft Agreement (
Agreement
) is a legal agreement between you (either an individual or a single entity) and Microsoft Corporation (
Microsoft
) for the version of the Microsoft specification identified above which you are about to download (
Specification
). BY DOWNLOADING, COPYING OR OTHERWISE USING THE SPECIFICATION, YOU AGREE TO BE BOUND BY THE TERMS OF THIS AGREEMENT. IF YOU DO NOT AGREE TO THE TERMS OF THIS AGREEMENT, DO NOT DOWNLOAD, COPY, OR USE THE SPECIFICATION.
The Specification is owned by Microsoft or its suppliers and is protected by copyright laws and international copyright treaties, as well as other intellectual property laws and treaties.
1. LIMITED LICENSE AND COVENANT NOT TO SUE.
(a) Provided that you comply with all terms and conditions of this Agreement and subject to the limitations in Sections 1(c) - (f) below, Microsoft grants to you the following non-exclusive, worldwide, royalty-free, non-transferable, non-sublicenseable license under any copyrights owned or licensable by Microsoft without payment of consideration to unaffiliated third parties, to reproduce the Specification solely for the purposes of creating portions of products which comply with the Specification in unmodified form.
(b) Provided that you comply with all terms and conditions of this Agreement and subject to the limitations in Sections 1(c) - (f) below, Microsoft grants to you the following non-exclusive, worldwide, royalty-free, non-transferable, non-sublicenseable, reciprocal limited covenant not to sue under its Necessary Claims solely to make, have made, use, import, and directly and indirectly, offer to sell, sell and otherwise distribute and dispose of portions of products which comply with the Specification in unmodified form.
For purposes of sections (a) and (b) above, the Specification is
unmodified
 if there are no changes, additions or extensions to the Specification, and
Necessary Claims
 means claims of a patent or patent application which are (1) owned or licenseable by Microsoft without payment of consideration to an unaffiliated third party; and (2) have an effective filing date on or before December 31, 2010, that must be infringed in order to make a portion(s) of a product that complies with the Specification. Necessary Claims does not include claims relating to semiconductor manufacturing technology or microprocessor circuits or claims not required to be infringed in complying with the Specification (even if in the same patent as Necessary Claims).
(c) The foregoing covenant not to sue shall not extend to any part or function of a product which (i) is not required to comply with the Specification in unmodified form, or (ii) to which there was a commercially reasonable alternative to infringing a Necessary Claim.
(d) Each of the license and the covenant not to sue described above shall be unavailable to you and shall terminate immediately if you or any of your Affiliates (collectively
Covenantee Party
Initiates
 any action for patent infringement against: (x) Microsoft or any of its Affiliates (collectively
Granting Party
), (y) any customers or distributors of the Granting Party, or other recipients of a covenant not to sue with respect to the Specification from the Granting Party (
Covenantees
); or (z) any customers or distributors of Covenantees (all parties identified in (y) and (z) collectively referred to as
Customers
), which action is based on a conformant implementation of the Specification. As used herein,
Affiliate
 means any entity which directly or indirectly controls, is controlled by, or is under common control with a party; and control shall mean the power, whether direct or indirect, to direct or cause the direction of the management or policies of any entity whether through the ownership of voting securities, by contract or otherwise.
Initiates
 means that a Covenantee Party is the first (as between the Granting Party and the Covenantee Party) to file or institute any legal or administrative claim or action for patent infringement against the Granting Party or any of the Customers.
Initiates
 includes any situation in which a Covenantee Party files or initiates a legal or administrative claim or action for patent infringement solely as a counterclaim or equivalent in response to a Granting Party first filing or instituting a legal or administrative patent infringement claim against such Covenantee Party.
(e) Each of the license and the covenant not to sue described above shall be conditioned on and limited to the sale, distribution, or other disposition of such compliant portions of products that are usable only by the firmware during the boot process and shall not extend to any purpose other than: (A) to create portions of an operating system (i) only as necessary to adapt such operating system so that it can directly interact with a firmware implementation of: Intel
s Extensible Firmware Initiative (EFI) Specification v. 1.0 and later, and the Unified Extensible Firmware Interface (UEFI) Forum
s UEFI Specifications v.2.0 and later  (together the
UEFI Specifications
); (ii) only as necessary to emulate an implementation of the UEFI Specifications; and (B) to create firmware, applications, utilities and/or drivers that will be used and/or licensed for only the following purposes: (i) to install, repair and maintain hardware, firmware and portions of operating system software which are utilized in the boot process; (ii) to provide to an operating system runtime services that are specified in the UEFI Specifications; (iii) to diagnose and correct failures in the hardware, firmware or operating system software; (iv) to query for identification of a computer system (whether by serial numbers, asset tags, user or otherwise); (v) to perform inventory of a computer system; and (vi) to manufacture, install and setup any hardware, firmware or operating system software.
(f) Microsoft reserves all other rights it may have in the Specification and any intellectual property therein. The furnishing of this document does not give you any license or covenant not to sue with respect to any other Microsoft patents, trademarks, copyrights or other intellectual property rights.
2. ADDITIONAL LIMITATIONS AND OBLIGATIONS.
(a)The foregoing license and covenant not to sue is applicable only to the version of the Specification which you are about to download. It does not apply to any additional versions of or extensions to the Specification.
(b)Without prejudice to any other rights, Microsoft may terminate this Agreement if you fail to comply with the terms and conditions of this Agreement. In such event you must destroy all copies of the Specification.
3. INTELLECTUAL PROPERTY RIGHTS. All ownership, title and intellectual property rights in and to the Specification are owned by Microsoft or its suppliers.
4. U.S. GOVERNMENT RIGHTS. Any Specification provided to the U.S. Government pursuant to solicitations issued on or after December 1, 1995 is provided with the commercial rights and restrictions described elsewhere herein. Any Specification provided to the U.S. Government pursuant to solicitations issued prior to December 1, 1995 is provided with RESTRICTED RIGHTS as provided for in FAR, 48 CFR 52.227-14 (JUNE 1987) or DFAR, 48 CFR 252.227-7013 (OCT 1988), as applicable.
5. EXPORT RESTRICTIONS. Export of the Specification, any part thereof, or any process or service that is the direct product of the Specification (the foregoing collectively referred to as the
Restricted Components
) from the United States is regulated by the Export Administration Regulations (EAR, 15 CFR 730-744) of the U.S. Commerce Department, Bureau of Export Administration (
). You agree to comply with the EAR in the export or re-export of the Restricted Components (i) to any country to which the U.S. has embargoed or restricted the export of goods or services, which currently include, but are not necessarily limited to Cuba, Iran, Iraq, Libya, North Korea, Sudan, Syria and the Federal Republic of Yugoslavia (including Serbia, but not Montenegro), or to any national of any such country, wherever located, who intends to transmit or transport the Restricted Components back to such country; (ii) to any person or entity who you know or have reason to know will utilize the Restricted Components in the design, development or production of nuclear, chemical or biological weapons; or (iii) to any person or entity who has been prohibited from participating in U.S. export transactions by any federal agency of the U.S. government. You warrant and represent that neither the BXA nor any other U.S. federal agency has suspended, revoked or denied your export privileges. For additional information see http://www.microsoft.com/exporting.
6. DISCLAIMER OF WARRANTIES. To the maximum extent permitted by applicable law, Microsoft and its suppliers provide the Specification (and all intellectual property therein) and any (if any) support services related to the Specification (
Support Services
) AS IS AND WITH ALL FAULTS, and hereby disclaim all warranties and conditions, either express, implied or statutory, including, but not limited to, any (if any) implied warranties or conditions of merchantability, of fitness for a particular purpose, of lack of viruses, of accuracy or completeness of responses, of results, and of lack of negligence or lack of workmanlike effort, all with regard to the Specification, any intellectual property therein and the provision of or failure to provide Support Services. ALSO, THERE IS NO WARRANTY OR CONDITION OF TITLE, QUIET ENJOYMENT, QUIET POSSESSION, CORRESPONDENCE TO DESCRIPTION OR NON-INFRINGEMENT, WITH REGARD TO THE SPECIFICATION AND ANY INTELLECTUAL PROPERTY THEREIN. THE ENTIRE RISK AS TO THE QUALITY OF OR ARISING OUT OF USE OR PERFORMANCE OF THE SPECIFICATION, ANY INTELLECTUAL PROPERTY THEREIN, AND SUPPORT SERVICES, IF ANY, REMAINS WITH YOU.
7. EXCLUSION OF INCIDENTAL, CONSEQUENTIAL AND CERTAIN OTHER DAMAGES. To the maximum extent permitted by applicable law, in no event shall Microsoft or its suppliers be liable for any special, incidental, indirect, or consequential damages whatsoever (including, but not limited to, damages for loss of profits or confidential or other information, for business interruption, for personal injury, for loss of privacy, for failure to meet any duty including of good faith or of reasonable care, for negligence, and for any other pecuniary or other loss whatsoever) arising out of or in any way related to the use of or inability to use the SPECIFICATION, ANY INTELLECTUAL PROPERTY THEREIN, the provision of or failure to provide Support Services, or otherwise under or in connection with any provision of this AGREEMENT, even in the event of the fault, tort (including negligence), strict liability, breach of contract or breach of warranty of Microsoft or any supplier, and even if Microsoft or any supplier has been advised of the possibility of such damages.
8. LIMITATION OF LIABILITY AND REMEDIES. Notwithstanding any damages that you might incur for any reason whatsoever (including, without limitation, all damages referenced above and all direct or general damages), the entire liability of Microsoft and any of its suppliers under any provision of this Agreement and your exclusive remedy for all of the foregoing shall be limited to the greater of the amount actually paid by you for the Specification or U.S.$5.00. The foregoing limitations, exclusions and disclaimers shall apply to the maximum extent permitted by applicable law, even if any remedy fails its essential purpose.
9. APPLICABLE LAW. If you acquired this Specification in the United States, this Agreement is governed by the laws of the State of Washington. If you acquired this Specification in Canada, unless expressly prohibited by local law, this Agreement is governed by the laws in force in the Province of Ontario, Canada; and, in respect of any dispute which may arise hereunder, you consent to the jurisdiction of the federal and provincial courts sitting in Toronto, Ontario. If this Specification was acquired outside the United States, then local law may apply.
10.QUESTIONS. Should you have any questions concerning this Agreement, or if you desire to contact Microsoft for any reason, please contact the Microsoft subsidiary serving your country, or write: Microsoft Sales Information Center/One Microsoft Way/Redmond, WA 98052-6399.
11.ENTIRE AGREEMENT. This Agreement is the entire agreement between you and Microsoft relating to the Specification and the Support Services (if any) and they supersede all prior or contemporaneous oral or written communications, proposals and representations with respect to the Specification or any other subject matter covered by this Agreement. To the extent the terms of any Microsoft policies or programs for Support Services conflict with the terms of this Agreement, the terms of this Agreement shall control.
Si vous avez acquis votre produit Microsoft au CANADA, la garantie limit
e suivante vous concerne :
RENONCIATION AUX GARANTIES. Dans toute la mesure permise par la l
gislation en vigueur, Microsoft et ses fournisseurs fournissent la Specification (et
 toute propri
 intellectuelle dans celle-ci) et tous (selon le cas) les services d
assistance li
 la Specification (
Services d
assistance
) TELS QUELS ET AVEC TOUS LEURS D
FAUTS, et par les pr
sentes excluent toute garantie ou condition, expresse ou implicite, l
gale ou conventionnelle,
crite ou verbale, y compris, mais sans limitation, toute (selon le cas) garantie ou condition implicite ou l
gale de qualit
 marchande, de conformit
 un usage particulier, d
absence de virus, d
exactitude et d
ponses, de r
sultats, d
efforts techniques et professionnels et d
absence de n
gligence, le tout relativement
 la Specification,
 toute propri
 intellectuelle dans celle-ci et
 la prestation ou
 la non-prestation des Services d
assistance. DE PLUS, IL N
Y A AUCUNE GARANTIE ET CONDITION DE TITRE, DE JOUISSANCE PAISIBLE, DE POSSESSION PAISIBLE, DE SIMILARIT
 LA DESCRIPTION ET D
ABSENCE DE CONTREFA
ON RELATIVEMENT
CIFICATION ET
 TOUTE PROPRI
 INTELLECTUELLE DANS CELLE-CI. VOUS SUPPORTEZ TOUS LES RISQUES D
COULANT DE L
UTILISATION ET DE LA PERFORMANCE DE LA SP
CIFICATION ET DE TOUTE PROPRI
 INTELLECTUELLE DANS CELLE-CI ET CEUX D
COULANT DES SERVICES D
ASSISTANCE (S
IL Y A LIEU).
EXCLUSION DES DOMMAGES INDIRECTS, ACCESSOIRES ET AUTRES. Dans toute la mesure permise par la l
gislation en vigueur, Microsoft et ses fournisseurs ne sont en aucun cas responsables de tout dommage sp
cial, indirect, accessoire, moral ou exemplaire quel qu
il soit (y compris, mais sans limitation, les dommages entra
s par la perte de b
fices ou la perte d
information confidentielle ou autre, l
interruption des affaires, les pr
judices corporels, la perte de confidentialit
faut de remplir toute obligation y compris les obligations de bonne foi et de diligence raisonnable, la n
gligence et toute autre perte p
cuniaire ou autre perte de quelque nature que ce soit) d
coulant de, ou de toute autre mani
utilisation ou l
impossibilit
utiliser la Sp
cification, toute propri
 intellectuelle dans celle-ci, la prestation ou la non-prestation des Services d
assistance ou autrement en vertu de ou relativement
 toute disposition de cette convention, que ce soit en cas de faute, de d
lit (y compris la n
gligence), de responsabilit
 stricte, de manquement
 un contrat ou de manquement
 une garantie de Microsoft ou de l
un de ses fournisseurs, et ce, m
me si Microsoft ou l
un de ses fournisseurs a
 de la possibilit
 de tels dommages.
LIMITATION DE RESPONSABILIT
 ET RECOURS. Malgr
 tout dommage que vous pourriez encourir pour quelque raison que ce soit (y compris, mais sans limitation, tous les dommages mentionn
s ci-dessus et tous les dommages directs et g
raux), la seule responsabilit
 de Microsoft et de ses fournisseurs en vertu de toute disposition de cette convention et votre unique recours en regard de tout ce qui pr
de sont limit
s au plus
 des montants suivants: soit (a) le montant que vous avez pay
 pour la Sp
cification, soit (b) un montant
quivalant
 cinq dollars U.S. (5,00 $ U.S.). Les limitations, exclusions et renonciations ci-dessus s
appliquent dans toute la mesure permise par la l
gislation en vigueur, et ce m
me si leur application a pour effet de priver un recours de son essence.
DROITS LIMIT
S DU GOUVERNEMENT AM
Tout Produit Logiciel fourni au gouvernement am
ricain conform
 des demandes
mises le ou apr
s le 1er d
cembre 1995 est offert avec les restrictions et droits commerciaux d
crits ailleurs dans la pr
sente convention. Tout Produit Logiciel fourni au gouvernement am
ricain conform
 des demandes
mises avant le 1er d
cembre 1995 est offert avec des DROITS LIMIT
S tels que pr
vus dans le FAR, 48CFR 52.227-14 (juin 1987) ou dans le FAR, 48CFR 252.227-7013 (octobre 1988), tels qu
applicables.
Sauf lorsqu
ment prohib
 par la l
gislation locale, la pr
sente convention est r
gie par les lois en vigueur dans la province d
Ontario, Canada. Pour tout diff
rend qui pourrait d
couler des pr
sentes, vous acceptez la comp
tence des tribunaux f
raux et provinciaux si
 Toronto, Ontario.
Si vous avez des questions concernant cette convention ou si vous d
sirez communiquer avec Microsoft pour quelque raison que ce soit, veuillez contacter la succursale Microsoft desservant votre pays, ou
: Microsoft Sales Information Center, One Microsoft Way, Redmond, Washington 98052-6399.
