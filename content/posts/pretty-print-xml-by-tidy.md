---
title: "tidy コマンドで XML を整形して出力する"
date: 2021-01-21T09:58:15+09:00
draft: false
toc: true
images:
tags: 
  - tidy
  - XML
---

XML をファイルとして保存して、 `-xml`, `-indent` オプションで整形できる

```bash
tidy -xml -indent ./myfile.xml
```

<!--more-->

## `-xml` オプション

- input するファイルのフォーマットが XML であることを指定


## `-indent` オプション

- 各要素をインデントして出力する


## manual

```zsh
TIDY(1)                                                                                                                                             TIDY(1)

NAME
       tidy - validate, correct, and pretty-print HTML files

SYNOPSIS
       tidy [option ...] [file ...] [option ...] [file ...]

DESCRIPTION
       Tidy  reads HTML, XHTML and XML files and writes cleaned up markup.  
       For HTML varians, it detects and corrects many common coding errors and strives to produce visually equivalent markup that is both W3C complaint and works on most browsers.  
       A common use of Tidy is to convert plain HTML to XHTML.  
       For generic XML files, Tidy is limited to correcting basic well-formedness errors and pretty printing.

       If no markup file is specified, Tidy reads the standard input.  
       If no output file is specified, Tidy writes markup to the standard output.  
       If no error file is specified, Tidy writes messages to the standard error.

OPTIONS
       Processing directives

       -indent or -i  to indent element content

       -omit          to omit optional end tags

       -wrap <column> to wrap text at the specified <column> (default is 68)

       -upper or -u   to force tags to upper case (default is lower case)

       -clean or -c   to replace FONT, NOBR and CENTER tags by CSS

       -bare or -b    to strip out smart quotes and em dashes, etc.

       -numeric or -n to output numeric rather than named entities

       -errors or -e  to only show errors

       -quiet or -q   to suppress nonessential output

       -xml           to specify the input is well formed XML

       -asxml         to convert HTML to well formed XHTML

       -asxhtml       to convert HTML to well formed XHTML

       -ashtml        to force XHTML to well formed HTML

       -access <level>
                      to do additional accessibility checks (<level> = 1, 2, 3)

       Character encodings

       -raw           to output values above 127 without conversion to entities

       -ascii         to use US-ASCII for output, ISO-8859-1 for input

       -latin1        to use ISO-8859-1 for both input and output

       -iso2022       to use ISO-2022 for both input and output

       -utf8          to use UTF-8 for both input and output

       -mac           to use MacRoman for input, US-ASCII for output

       -utf16le       to use UTF-16LE for both input and output

       -utf16be       to use UTF-16BE for both input and output

       -utf16         to use UTF-16 for both input and output

       -win1252       to use Windows-1252 for input, US-ASCII for output

       -big5          to use Big5 for both input and output

       -shiftjis      to use Shift_JIS for both input and output

       -language <lang>
                      to set the two-letter language code <lang> (for future use)

       File manipulation

       -output or -o <file>
                      to write output to the specified <file>

       -f <file>      to write errors to the specified <file>

       -config <file> to set configuration options from the specified <file>

       -modify or -m  to modify the original input files

       Miscellaneous

       -version or -v to show the version of Tidy

       -help, -h or -?
                      to list the command line options

       -help-config   to list all configuration options

       -show-config   to list the current configuration settings


USAGE
       Use --blah blarg for any configuration option "blah" with argument "blarg"

       Input/Output default to stdin/stdout respectively Single letter options apart from -f and -o may be combined as in:  tidy -f errs.txt -imu  foo.html
       For further info on HTML see http://www.w3.org/MarkUp

       For more information about HTML Tidy, visit the project home page at http://tidy.sourceforge.net.  Here, you will find links to documentation, mail-
       ing lists (with searchable archives) and links to report bugs.

ENVIRONMENT
       HTML_TIDY      Name of the default configuration file.  This should be an absolute path, since you will probably invoke tidy from different directo-
                      ries.   The value of HTML_TIDY will be parsed after the compiled-in default (defined with -DCONFIG_FILE), but before any of the files
                      specified using -config.

EXIT STATUS
       0      All input files were processed successfully.

       1      There were warnings.

       2      There were errors.

SEE ALSO
       HTML Tidy Project Page at http://tidy.sourceforge.net

       Dave Raggett's Tidy Overview at http://www.w3.org/People/Raggett/tidy/

       Tidy Quick Reference at http://tidy.sourceforge.net/docs/quickref.html

       For information about TidyLib, see http://tidy.sourceforge.net/libintro.html

AUTHORS
       Dave Raggett <dsr@w3.org>.

       Terry Teague <terry_teague@users.sourceforge.net>.

       Bjoern Hoehrmann <bjoern@hoehrmann.de>

       Charles Reitzel <creitzel@rcn.com>

       This manual page was written by Matej Vela <vela@debian.org> and updated by Charles Reitzel.

                                                                      December 1, 2002                                                              TIDY(1)
```


## 実行例

- https://samltool.io/ で公開されている SAML レスポンスを `saml.xml` として保存

```zsh
% tidy -xml -indent ./saml.xml 
No warnings or errors were found.

<samlp:Response xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
ID="_621c4c4ae5d60c768cc2" Version="2.0"
IssueInstant="2014-10-14T14:32:17Z"
Destination="https://app.auth0.com/tester/samlp">
  <saml:Issuer xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">
  urn:matugit.auth0.com</saml:Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
  </samlp:Status>
  <saml:Assertion xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
  Version="2.0" ID="_5VK7LT7FliUkkaQuW6r4brF0DG5E3X76"
  IssueInstant="2014-10-14T14:32:17.251Z">
    <saml:Issuer>urn:matugit.auth0.com</saml:Issuer>
    <Signature xmlns="http://www.w3.org/2000/09/xmldsig#">
      <SignedInfo>
        <CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
        <SignatureMethod Algorithm="http://www.w3.org/2000/09/xmldsig#rsa-sha1" />
        <Reference URI="#_5VK7LT7FliUkkaQuW6r4brF0DG5E3X76">
          <Transforms>
            <Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature" />
            <Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
          </Transforms>
          <DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1" />
          <DigestValue>ZDkfGO3H1Tu50hawzQVjsACzJwc=</DigestValue>
        </Reference>
      </SignedInfo>
      <SignatureValue>
      1Fgpt7AaHcME2gTA158achvGQVqDwHSHsHF3/a5s7djeO1AaZ84Gz5eiWD+cdIz6hoT1j2v/7Qtfj5bsMNxulBnSzfL4TT4++AK/zFO4ad2mwABFjMMNiogwwT3tzy3aRwjgSfl3VKESoz3Zp8KL/JNm/TRKz95TD7H6WnfKZoLwErGzNw6gVs1y9XYxIEg46GzUb07g23TFmrv3wHlx2TpKUN/ne4Z28KAQzXqVyykJVaKQ/gbBNC/8AQKlol8fLGSheOKQ0vgEE1vFnVVCEmp30YapdKeWW2qcqHb7Oqdm+9b2mOUkqbaxH5ixBbYaqZaQCt5WF4P5BxnMe4Bp8w==</SignatureValue>
      <KeyInfo>
        <X509Data>
          <X509Certificate>
          MIIDODCCAiCgAwIBAgIJAN7O9hef05RJMA0GCSqGSIb3DQEBBQUAMBwxGjAYBgNVBAMTEW1hdHVnaXQuYXV0aDAuY29tMB4XDTEzMDYxNjEzMjQwNloXDTI3MDIyMzEzMjQwNlowHDEaMBgGA1UEAxMRbWF0dWdpdC5hdXRoMC5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQD8kEfEcLiyMK0q4E4xT/mglcXsf9oCWdTjB0vswFXQPCDgUk5daJmO4oRrCIuWgtChDyMc5M5MTexqrCntQlKpnexkzDxCvPX/IRJstjJDSH+WXljRYDbXTgAZAjjjAKqDBIeUWaw1FcUy3SOmQVtyUdfjXNqLzxKS8SUWYY3Ia6PT/uoUjUFmUuJEFpBlB79GW+Tv/ZidY53gHyhEYOBzm1vKZS1vtPfxxpBqu7Vkf+5lq3RYjuoVUYN03TIJveaa7BBL0je8z7UvvOGgrGuodSKHneq7ueex4OCscCJZbK7MlfbmFiuSniVCg/yMrx/okDme38sOXH1A/zJu2lYpAgMBAAGjfTB7MB0GA1UdDgQWBBQL3cuDXNXu4USrpv44dzfGyHIXDTBMBgNVHSMERTBDgBQL3cuDXNXu4USrpv44dzfGyHIXDaEgpB4wHDEaMBgGA1UEAxMRbWF0dWdpdC5hdXRoMC5jb22CCQDezvYXn9OUSTAMBgNVHRMEBTADAQH/MA0GCSqGSIb3DQEBBQUAA4IBAQCAtAmKHoX7OerzFUQ0Ojrg9C1uzT6fOLIlXyaDswk2tLE6ftFhSiVzT1TtgTFv2o6IA47tW6bqL7t6Z5JN/L8cno+kxeKOgPz2CvbUTJmtROvFV/TDkFsYmFJZ8+6nZOAtXRZPWFpa6KE4Lc5+7J94sX2AiYPFCnXRvZGMhJ6qtc+jh54QEIlxZtkxWUBGs2fhOQm+Ux0uy1qxSzhp3lmvNa79KdvmQirSitDovyidalpwX750WGNfQoISe94e3clWa4uKXgZ7AYmhd1w4kNs7seFdtvCcqQJrKYT2cBTINztL6SeqeF/xC//lGDo9UJic4pcrqj1P3WGHYIslOWBQ</X509Certificate>
        </X509Data>
      </KeyInfo>
    </Signature>
    <saml:Subject>
      <saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified">
      github|175880</saml:NameID>
      <saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">

        <saml:SubjectConfirmationData NotOnOrAfter="2014-10-14T15:32:17.251Z"
        Recipient="https://app.auth0.com/tester/samlp" />
      </saml:SubjectConfirmation>
    </saml:Subject>
    <saml:Conditions NotBefore="2014-10-14T14:32:17.251Z"
    NotOnOrAfter="2014-10-14T15:32:17.251Z">
      <saml:AudienceRestriction>
        <saml:Audience>urn:foo</saml:Audience>
      </saml:AudienceRestriction>
    </saml:Conditions>
    <saml:AttributeStatement xmlns:xs="http://www.w3.org/2001/XMLSchema"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <saml:Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier">

        <saml:AttributeValue xsi:type="xs:anyType">
        github|175880</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress">

        <saml:AttributeValue xsi:type="xs:anyType">
        matiasw@gmail.com</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">

        <saml:AttributeValue xsi:type="xs:anyType">Matias
        Woloski</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn">

        <saml:AttributeValue xsi:type="xs:anyType">
        matiasw@gmail.com</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.auth0.com/identities/default/access_token">

        <saml:AttributeValue xsi:type="xs:anyType">
        3a7d0dfeffe12812c37112daa830abef570089b4</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.auth0.com/identities/default/provider">

        <saml:AttributeValue xsi:type="xs:anyType">
        github</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.auth0.com/identities/default/connection">

        <saml:AttributeValue xsi:type="xs:anyType">
        github</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.auth0.com/identities/default/isSocial">

        <saml:AttributeValue xsi:type="xs:anyType">
        true</saml:AttributeValue>
      </saml:Attribute>
    </saml:AttributeStatement>
    <saml:AuthnStatement AuthnInstant="2014-10-14T14:32:17.251Z">
      <saml:AuthnContext>
        <saml:AuthnContextClassRef>
        urn:oasis:names:tc:SAML:2.0:ac:classes:unspecified</saml:AuthnContextClassRef>
      </saml:AuthnContext>
    </saml:AuthnStatement>
  </saml:Assertion>
</samlp:Response>

To learn more about HTML Tidy see http://tidy.sourceforge.net
Please send bug reports to html-tidy@w3.org
HTML and CSS specifications are available from http://www.w3.org/
Lobby your company to join W3C, see http://www.w3.org/Consortium
```


## この記事を試した環境

```zsh
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% tidy -version 
HTML Tidy for Mac OS X released on 31 October 2006 - Apple Inc. build 17.2
```