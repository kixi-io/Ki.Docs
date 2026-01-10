---
title: KiTZ Time Zones
nav_order: 6
---

# Ki Time Zone Specification

The Ki Time Zone (KiTZ) specification defines short, easily readable, and unique identifiers for common time zone names such as `US/PST` for US Pacific Standard Time and `JP/JST` for Japan Standard Time. Ki Time Zones are disambiguated with a two letter country code prefix. This is necessary for time zone names such as `CST`, which can mean China Standard Time, US Central Standard Time, Canadian Central Standard Time, or Australian Central Standard Time. Their Ki Time Zones are, respectively, `CN/CST`, `US/CST`, `CA/CST`, and `AU/CST`.

**Q: Why not use Time Zone IDs from the [IANA time zone database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)?**

A: The IANA Time Zone DB is a comprehensive database, including historical time zones, that is useful for many purposes. That being said, it is cumbersome for many practical applications because the IDs are _long_, _heavily redundant_ (multiple IDs for the same offsets in the same country), and aren't what people actually use when they speak or write. For example, everyone in Los Angeles, California (US) will tell you they are in the `PST` time zone, but the IANA timezone ID (formatted for ISO-8601 date/time) is `[America/Los_Angeles]`. Los Angeles's KiTZ, `US/PST`, is short, immediately recognizable and unambiguous. It applies to the entire US Pacific Standard Time Zone. During daylight savings time, it is US/PDT. Similarly, the IANA timezone ID for Buenos Aires is [America/Argentina/Buenos_Aires]. Argentina has _one time zone_ but 13 IANA time zone IDs, all with the same `UTC-3` offset. The KiTZ for Argentina is `AR/ART`. It applies to the entire country, is short, unambiguous, and easily recognizable as "Argentina Time" (ART is the common name for Argentina's time zone.)

Ki Time Zones are not yet comprehensive. We are working on adding all countries. See the list below. 

The KiTZ spec does not capture historical time zones existing before January 1, 2020. Using UTC offsets is recommended for legacy time zones. 

| KiTZ | Country | UTC Offset |
| --- | ------- | ---------- |
| AR/ART  | Argentina | -3 |  
| AU/ACST | Australia | +9:30 | 
| AU/ACDT | Australia | +10:30 | 
| AU/AEST | Australia | +10 | 
| AU/AEDT | Australia | +11 |
| AU/AWST | Australia | +8 |
| AU/NFT  | Australia | +11 |
| AU/NFDT | Australia | +12 |
| BR/ACT  | Brazil | -5 |
| BR/AMT  | Brazil | -4 |
| BR/BRT  | Brazil | -3 |
| BR/FNT  | Brazil | -2 |
| CA/AST  | Canada | -4 |
| CA/ADT  | Canada | -3 |
| CA/CST  | Canada | -6 |
| CA/CDT  | Canada | -5 |
| CA/EST  | Canada | -5 |
| CA/EDT  | Canada | -4 |
| CA/MST  | Canada | -7 |
| CA/MDT  | Canada | -6 |
| CA/NST  | Canada | -3:30 |
| CA/NDT  | Canada | -2:30 |
| CA/PST  | Canada | -8 |
| CA/PDT  | Canada | -7 |
| CN/CST  | China | +8 |
| CO/COT  | Columbia | -5 |
| CZ/CET  | Czech Republic | +1 |
| CZ/CEST | Czech Republic | +2 |
| EG/EET  | Egypt | +2 |
| ET/EAT  | Ethiopia | +3 |
| FR/CET  | France | +1 |
| FR/CEST | France | +2 |
| DE/CET  | Germany | +1 |
| DE/CEST | Germany | +2 |
| GR/EET  | Greece | +2 |
| GR/EEST | Greece | +3 |
| IN/IST  | India| +5:30 |
| ID/WIB  | Indonesia | +7 |
| ID/WITA | Indonesia | +8 |
| ID/WIT  | Indonesia | +9 |
| IR/IRST | Iran | +3:30 |
| IR/IRDT | Iran | +4:30 |
| IQ/AST  | Iraq | +3 |
| IE/IST  | Ireland | +1 |
| IL/IST  | Israel | +2 |
| IL/IDT  | Israel | +3 |
| IT/CET  | Italy | +1 |
| IT/CEST | Italy | +2 |
| JP/JST  | Japan | +9 |
| KE/EAT  | Kenya | +3 |
| LU/CET | Luxembourg | +1 |
| MX/CST  | Mexico | -6 |
| MX/CDT  | Mexico | -5 |
| MX/EST  | Mexico | -5 |
| MX/MST  | Mexico | -7 |
| MX/MDT  | Mexico | -6 |
| MX/PST  | Mexico | -8 |
| MX/PDT  | Mexico | -7 |
| NL/CET  | Netherlands | +1 |
| NL/CEST | Netherlands | +2 |
| NL/AST  | Netherlands | -4 |
| NZ/NZST | New Zealand | +12 |
| NZ/NZDT | New Zealand | +13 |
| NG/WAT  | Nigeria | +1 |
| PK/PST  | Pakistan | +5 |
| PL/CET  | Poland | +1 |
| PL/CEST | Poland | +2 |
| RO/EET  | Romania | +2 |
| RO/EEST | Romania | +3 |
| RU/KALT | Russia | +2 |
| RU/MSK  | Russia | +3 |
| RU/SAMT | Russia | +4 |
| RU/YEKT | Russia | +5 |
| RU/OMST | Russia | +6 |
| RU/KRAT | Russia | +7 |
| RU/IRKT | Russia | +8 |
| RU/YAKT | Russia | +9 |
| RU/VLAT | Russia | +10 |
| RU/MAGT | Russia | +11 |
| RU/PETT | Russia | +12 |
| SA/AST  | Saudi Arabia | +3 |
| SG/SST  | Singapore | +8 |
| ZA/SAST | South Africa | +2 |
| KR/KST  | South Korea | +9 |
| ES/CET  | Spain | +1 |
| ES/CEST | Spain | +2 |
| SE/CET  | Sweden | +1 |
| SE/CEST | Sweden | +2 |
| CH/CET  | Switzerland | +1 |
| CH/CEST | Switzerland | +2 |
| TH/ICT  | Thailand | +7 |
| TR/FET  | Turkey | +3 |
| UA/EET  | Ukraine | +2 |
| UA/EEST | Ukraine | +3 |
| UTC     | United Kingdom | 0 |
| GB/DST  | United Kingdom | +1 |
| US/AST  | United States | -4 |
| US/AKST | United States | -9 |
| US/AKDT | United States | -8 |
| US/CHST | United States | +10
| US/CST  | United States | −6 |
| US/CDT  | United States | -5 |
| US/EST  | United States | -5 |
| US/EDT  | United States | -4 |
| US/HST  | United States | -10 |
| US/MST  | United States | -7 |
| US/MDT  | United States | -6 |
| US/PST  | United States | -8 |
| US/PDT  | United States | -7 |
| US/SST  | United States | -11 |
| UY/UYT  | Uruguay | -3 |
| VE/VET  | Venezuela | -4 |
| VN/ICT  | Vietnam | +7 |
