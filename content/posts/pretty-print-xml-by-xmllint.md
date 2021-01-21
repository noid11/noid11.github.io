---
title: "xmllint コマンドで XML を整形して出力する"
date: 2021-01-21T10:26:45+09:00
draft: false
toc: true
images:
tags: 
  - XML
  - xmllint
---

XML をファイルとして保存して、 `--format` オプションで整形できる

```bash
xmllint --format ./myfile.xml
```

<!--more-->

## manual

```bash
XMLLINT(1)                                                             xmllint Manual                                                            XMLLINT(1)

NAME
       xmllint - command line XML tool

SYNOPSIS
       xmllint [--version | --debug | --shell | --xpath "XPath_expression" | --debugent | --copy | --recover | --noent | --noout | --nonet |
               --path "PATH(S)" | --load-trace | --htmlout | --nowrap | --valid | --postvalid | --dtdvalid URL | --dtdvalidfpi FPI | --timing |
               --output FILE | --repeat | --insert | --compress | --html | --xmlout | --push | --memory | --maxmem NBBYTES | --nowarning | --noblanks |
               --nocdata | --format | --encode ENCODING | --dropdtd | --nsclean | --testIO | --catalogs | --nocatalogs | --auto | --xinclude |
               --noxincludenode | --loaddtd | --dtdattr | --stream | --walker | --pattern PATTERNVALUE | --chkregister | --relaxng SCHEMA | --schema SCHEMA
               | --c14n] {XML-FILE(S)... | -}

       xmllint --help

DESCRIPTION
       The xmllint program parses one or more XML files, specified on the command line as XML-FILE (or the standard input if the filename provided is - ).
       It prints various types of output, depending upon the options selected. It is useful for detecting errors both in XML code and in the XML parser itself.

       xmllint is included in libxml(3).

OPTIONS
       xmllint accepts the following options (in alphabetical order):

       --auto
           Generate a small document for testing purposes.

       --catalogs
           Use the SGML catalog(s) from SGML_CATALOG_FILES. Otherwise XML catalogs starting from /etc/xml/catalog are used by default.

       --chkregister
           Turn on node registration. Useful for developers testing libxml(3) node tracking code.

       --compress
           Turn on gzip(1) compression of output.

       --copy
           Test the internal copy implementation.

       --c14n
           Use the W3C XML Canonicalisation (C14N) to serialize the result of parsing to stdout. It keeps comments in the result.

       --dtdvalid URL
           Use the DTD specified by an URL for validation.

       --dtdvalidfpi FPI
           Use the DTD specified by a Formal Public Identifier FPI for validation, note that this will require a catalog exporting that Formal Public Identifier to work.

       --debug
           Parse a file and output an annotated tree of the in-memory version of the document.

       --debugent
           Debug the entities defined in the document.

       --dropdtd
           Remove DTD from output.

       --dtdattr
           Fetch external DTD and populate the tree with inherited attributes.

       --encode ENCODING
           Output in the given encoding. Note that this works for full document not fragments or result from XPath queries.

       --format
           Reformat and reindent the output. The XMLLINT_INDENT environment variable controls the indentation. The default value is two spaces " ").

       --help
           Print out a short usage summary for xmllint.

       --html
           Use the HTML parser.

       --htmlout
           Output results as an HTML file. 
           This causes xmllint to output the necessary HTML tags surrounding the result tree output so the results can be displayed/viewed in a browser.

       --insert
           Test for valid insertions.

       --loaddtd
           Fetch an external DTD.

       --load-trace
           Display all the documents loaded during the processing to stderr.

       --maxmem NNBYTES
           Test the parser memory support.  
           NNBYTES is the maximum number of bytes the library is allowed to allocate. 
           This can also be used to make sure batch processing of XML files will not exhaust the virtual memory of the server running them.

       --memory
           Parse from memory.

       --noblanks
           Drop ignorable blank spaces.

       --nocatalogs
           Do not use any catalogs.

       --nocdata
           Substitute CDATA section by equivalent text nodes.

       --noent
           Substitute entity values for entity references. By default, xmllint leaves entity references in place.

       --nonet
           Do not use the Internet to fetch DTDs or entities.

       --noout
           Suppress output. By default, xmllint outputs the result tree.

       --nowarning
           Do not emit warnings from the parser and/or validator.

       --nowrap
           Do not output HTML doc wrapper.

       --noxincludenode
           Do XInclude processing but do not generate XInclude start and end nodes.

       --nsclean
           Remove redundant namespace declarations.

       --output FILE
           Define a file path where xmllint will save the result of parsing. 
           Usually the programs build a tree and save it on stdout, with this option the result XML instance will be saved onto a file.

       --path "PATH(S)"
           Use the (space- or colon-separated) list of filesystem paths specified by PATHS to load DTDs or entities. 
           Enclose space-separated lists by quotation marks.

       --pattern PATTERNVALUE
           Used to exercise the pattern recognition engine, which can be used with the reader interface to the parser. 
           It allows to select some nodes in the document based on an XPath (subset) expression. Used for debugging.

       --postvalid
           Validate after parsing has completed.

       --push
           Use the push mode of the parser.

       --recover
           Output any parsable portions of an invalid document.

       --relaxng SCHEMA
           Use RelaxNG file named SCHEMA for validation.

       --repeat
           Repeat 100 times, for timing or profiling.

       --schema SCHEMA
           Use a W3C XML Schema file named SCHEMA for validation.

       --shell
           Run a navigating shell. Details on available commands in shell mode are below (see the section called "SHELL COMMANDS").

       --xpath "XPath_expression"
           Run an XPath expression given as argument and print the result. 
           In case of a nodeset result, each node in the node set is serialized in full in the output. 
           In case of an empty node set the "XPath set is empty" result will be shown and an error exit code will be returned.

       --stream
           Use streaming API - useful when used in combination with --relaxng or --valid options for validation of files that are too large to be held in memory.

       --testIO
           Test user input/output support.

       --timing
           Output information about the time it takes xmllint to perform the various steps.

       --valid
           Determine if the document is a valid instance of the included Document Type Definition (DTD). 
           A DTD to be validated against also can be specified at the command line using the --dtdvalid option. 
           By default, xmllint also checks to determine if the document is well-formed.

       --version
           Display the version of libxml(3) used.

       --walker
           Test the walker module, which is a reader interface but for a document tree, instead of using the reader API on an unparsed document it works on an existing in-memory tree. 
           Used for debugging.

       --xinclude
           Do XInclude processing.

       --xmlout
           Used in conjunction with --html. 
           Usually when HTML is parsed the document is saved with the HTML serializer. 
           But with this option the resulting document is saved with the XML serializer. 
           This is primarily used to generate XHTML from HTML input.

SHELL COMMANDS
       xmllint offers an interactive shell mode invoked with the --shell command. Available commands in shell mode include (in alphabetical order):

       base
           Display XML base of the node.

       bye
           Leave the shell.

       cat NODE
           Display the given node or the current one.

       cd PATH
           Change the current node to the given path (if unique) or root if no argument is given.

       dir PATH
           Dumps information about the node (namespace, attributes, content).

       du PATH
           Show the structure of the subtree under the given path or the current node.

       exit
           Leave the shell.

       help
           Show this help.

       free
           Display memory usage.

       load FILENAME
           Load a new document with the given filename.

       ls PATH
           List contents of the given path or the current directory.

       pwd
           Display the path to the current node.

       quit
           Leave the shell.

       save FILENAME
           Save the current document to the given filename or to the original name.

       validate
           Check the document for errors.

       write FILENAME
           Write the current node to the given filename.

ENVIRONMENT
       SGML_CATALOG_FILES
           SGML catalog behavior can be changed by redirecting queries to the user's own set of catalogs. 
           This can be done by setting the SGML_CATALOG_FILES environment variable to a list of catalogs. 
           An empty one should deactivate loading the default /etc/sgml/catalog catalog.

       XML_CATALOG_FILES
           XML catalog behavior can be changed by redirecting queries to the user's own set of catalogs. 
           This can be done by setting the XML_CATALOG_FILES environment variable to a list of catalogs. 
           An empty one should deactivate loading the default /etc/xml/catalog catalog.

       XML_DEBUG_CATALOG
           Setting the environment variable XML_DEBUG_CATALOG to non-zero using the export command outputs debugging information related to catalog operations.

       XMLLINT_INDENT
           Setting the environment variable XMLLINT_INDENT controls the indentation. The default value is two spaces " ".

DIAGNOSTICS
       xmllint return codes provide information that can be used when calling it from scripts.

       0
           No error

       1
           Unclassified

       2
           Error in DTD

       3
           Validation error

       4
           Validation error

       5
           Error in schema compilation

       6
           Error writing output

       7
           Error in pattern (generated when --pattern option is used)

       8
           Error in Reader registration (generated when --chkregister option is used)

       9
           Out of memory error

SEE ALSO
       libxml(3)

       More information can be found at

       o   libxml(3) web page http://www.xmlsoft.org/

       o   W3C XSLT page http://www.w3.org/TR/xslt

AUTHORS
       John Fleck <jfleck@inkstain.net>
           Author.

       Ziying Sherwin <sherwin@nlm.nih.gov>
           Author.

       Heiko Rupp <hwr@pilhuhn.de>
           Author.

COPYRIGHT
       Copyright (C) 2001, 2004

libxml2                                                                    $Date$                                                                XMLLINT(1)
```


## 実行例

- https://samltool.io/ で公開されている SAML レスポンスを `saml.xml` として保存

```zsh
% xmllint --format ./saml.xml 
<?xml version="1.0"?>
<samlp:Response xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" ID="_621c4c4ae5d60c768cc2" Version="2.0" IssueInstant="2014-10-14T14:32:17Z" Destination="https://app.auth0.com/tester/samlp">
  <saml:Issuer xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">urn:matugit.auth0.com</saml:Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success"/>
  </samlp:Status>
  <saml:Assertion xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" Version="2.0" ID="_5VK7LT7FliUkkaQuW6r4brF0DG5E3X76" IssueInstant="2014-10-14T14:32:17.251Z">
    <saml:Issuer>urn:matugit.auth0.com</saml:Issuer>
    <Signature xmlns="http://www.w3.org/2000/09/xmldsig#">
      <SignedInfo>
        <CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
        <SignatureMethod Algorithm="http://www.w3.org/2000/09/xmldsig#rsa-sha1"/>
        <Reference URI="#_5VK7LT7FliUkkaQuW6r4brF0DG5E3X76">
          <Transforms>
            <Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
            <Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
          </Transforms>
          <DigestMethod Algorithm="http://www.w3.org/2000/09/xmldsig#sha1"/>
          <DigestValue>ZDkfGO3H1Tu50hawzQVjsACzJwc=</DigestValue>
        </Reference>
      </SignedInfo>
      <SignatureValue>1Fgpt7AaHcME2gTA158achvGQVqDwHSHsHF3/a5s7djeO1AaZ84Gz5eiWD+cdIz6hoT1j2v/7Qtfj5bsMNxulBnSzfL4TT4++AK/zFO4ad2mwABFjMMNiogwwT3tzy3aRwjgSfl3VKESoz3Zp8KL/JNm/TRKz95TD7H6WnfKZoLwErGzNw6gVs1y9XYxIEg46GzUb07g23TFmrv3wHlx2TpKUN/ne4Z28KAQzXqVyykJVaKQ/gbBNC/8AQKlol8fLGSheOKQ0vgEE1vFnVVCEmp30YapdKeWW2qcqHb7Oqdm+9b2mOUkqbaxH5ixBbYaqZaQCt5WF4P5BxnMe4Bp8w==</SignatureValue>
      <KeyInfo>
        <X509Data>
          <X509Certificate>MIIDODCCAiCgAwIBAgIJAN7O9hef05RJMA0GCSqGSIb3DQEBBQUAMBwxGjAYBgNVBAMTEW1hdHVnaXQuYXV0aDAuY29tMB4XDTEzMDYxNjEzMjQwNloXDTI3MDIyMzEzMjQwNlowHDEaMBgGA1UEAxMRbWF0dWdpdC5hdXRoMC5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQD8kEfEcLiyMK0q4E4xT/mglcXsf9oCWdTjB0vswFXQPCDgUk5daJmO4oRrCIuWgtChDyMc5M5MTexqrCntQlKpnexkzDxCvPX/IRJstjJDSH+WXljRYDbXTgAZAjjjAKqDBIeUWaw1FcUy3SOmQVtyUdfjXNqLzxKS8SUWYY3Ia6PT/uoUjUFmUuJEFpBlB79GW+Tv/ZidY53gHyhEYOBzm1vKZS1vtPfxxpBqu7Vkf+5lq3RYjuoVUYN03TIJveaa7BBL0je8z7UvvOGgrGuodSKHneq7ueex4OCscCJZbK7MlfbmFiuSniVCg/yMrx/okDme38sOXH1A/zJu2lYpAgMBAAGjfTB7MB0GA1UdDgQWBBQL3cuDXNXu4USrpv44dzfGyHIXDTBMBgNVHSMERTBDgBQL3cuDXNXu4USrpv44dzfGyHIXDaEgpB4wHDEaMBgGA1UEAxMRbWF0dWdpdC5hdXRoMC5jb22CCQDezvYXn9OUSTAMBgNVHRMEBTADAQH/MA0GCSqGSIb3DQEBBQUAA4IBAQCAtAmKHoX7OerzFUQ0Ojrg9C1uzT6fOLIlXyaDswk2tLE6ftFhSiVzT1TtgTFv2o6IA47tW6bqL7t6Z5JN/L8cno+kxeKOgPz2CvbUTJmtROvFV/TDkFsYmFJZ8+6nZOAtXRZPWFpa6KE4Lc5+7J94sX2AiYPFCnXRvZGMhJ6qtc+jh54QEIlxZtkxWUBGs2fhOQm+Ux0uy1qxSzhp3lmvNa79KdvmQirSitDovyidalpwX750WGNfQoISe94e3clWa4uKXgZ7AYmhd1w4kNs7seFdtvCcqQJrKYT2cBTINztL6SeqeF/xC//lGDo9UJic4pcrqj1P3WGHYIslOWBQ</X509Certificate>
        </X509Data>
      </KeyInfo>
    </Signature>
    <saml:Subject>
      <saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified">github|175880</saml:NameID>
      <saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <saml:SubjectConfirmationData NotOnOrAfter="2014-10-14T15:32:17.251Z" Recipient="https://app.auth0.com/tester/samlp"/>
      </saml:SubjectConfirmation>
    </saml:Subject>
    <saml:Conditions NotBefore="2014-10-14T14:32:17.251Z" NotOnOrAfter="2014-10-14T15:32:17.251Z">
      <saml:AudienceRestriction>
        <saml:Audience>urn:foo</saml:Audience>
      </saml:AudienceRestriction>
    </saml:Conditions>
    <saml:AttributeStatement xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <saml:Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier">
        <saml:AttributeValue xsi:type="xs:anyType">github|175880</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress">
        <saml:AttributeValue xsi:type="xs:anyType">matiasw@gmail.com</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
        <saml:AttributeValue xsi:type="xs:anyType">Matias Woloski</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn">
        <saml:AttributeValue xsi:type="xs:anyType">matiasw@gmail.com</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.auth0.com/identities/default/access_token">
        <saml:AttributeValue xsi:type="xs:anyType">3a7d0dfeffe12812c37112daa830abef570089b4</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.auth0.com/identities/default/provider">
        <saml:AttributeValue xsi:type="xs:anyType">github</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.auth0.com/identities/default/connection">
        <saml:AttributeValue xsi:type="xs:anyType">github</saml:AttributeValue>
      </saml:Attribute>
      <saml:Attribute Name="http://schemas.auth0.com/identities/default/isSocial">
        <saml:AttributeValue xsi:type="xs:anyType">true</saml:AttributeValue>
      </saml:Attribute>
    </saml:AttributeStatement>
    <saml:AuthnStatement AuthnInstant="2014-10-14T14:32:17.251Z">
      <saml:AuthnContext>
        <saml:AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:unspecified</saml:AuthnContextClassRef>
      </saml:AuthnContext>
    </saml:AuthnStatement>
  </saml:Assertion>
</samlp:Response>
```


## この記事を試した環境

```zsh
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% xmllint --version
xmllint: using libxml version 20904
   compiled with: Threads Tree Output Push Reader Patterns Writer SAXv1 FTP HTTP DTDValid HTML Legacy C14N Catalog XPath XPointer XInclude ICU ISO8859X Unicode Regexps Automata Expr Schemas Schematron Modules Debug Zlib 
```