## Notes ##

This folder includes examples for how to log using the Graylog Extended Log Format (GELF)

### GELF Advantages ###

* Syslog is generally limited 1024 byte messages and doesn't cope well with newlines or special characters embedded within the log message. GELF is practically unlimited; it even supports chunking if messages greater than 8192 bytes are to be sent over UDP.
* Syslog is unstructured and requires parsing using GROK patterns. GELF is structured as a JSON message and can be parsed in a standard way which is both efficient and requires minimal setup.

### GELF Log Format ### 

JSON key/value string terminated by a NULL (NUL/NOP) byte.

Minimal JSON parameters:
* version (string: "1.1")
* host (string: "whatever hostname string you like")
* short_message (string: "a shortish message")
* full_message (string: "a longer message")
* level (number: 0-7 (same as standard syslog severity level codes))

Additional JSON key/value pairs should be prefixed with a single underscore ("_") character; these will in turn be created as key/value pairs within the indexed log. Strings or numbers only; don't quote numbers if you want them interpretted as numbers.

### iRule Specific Stuff ###

#### Timestamp Accuracy ####

While you can get away with simply dumping in the output from [clock seconds]; it's not terribly accurate and your logs may be processed/displayed out of order.

Use the following to generate seconds since epoch with some millisecond level accuracy.

`set clicks [clock clicks -milliseconds]
set http_request_time [string range ${clicks} 0 end-3].[string range ${clicks} end-2 end]
unset clicks`

#### NULL Byte Terminator ####

NULL can normally be expressed as \0 or \x00; however these WILL not work in iRules. The TCL parser will reinterpret them and re-encode them.

You *MUST* use a second HSL:: send statement with only the following for content:

`[binary format c* { 0 }]`

Do not place the binary format command within a string, the TCL parser will STILL reinterpret and re-encode the characters.

#### Optional Stuff ####

Replace or escape characters with special significance.

Even though GELF format logs, when NULL terminated, can include carriage returns and newline characters in their raw format; we don't necessarily want them in the final log.

Similarly with double quote characters; we don't necessarily want them in the final log.
Use the TCL string map function to remap any characters or strings that should be converted to something more sensible.
e.g. to replace CR/LF, or CR and LF on their own, plus double quotes, use the following:

`[string map {"\r\n" {\r\n} "\r" {\r} "\n" {\n} {"} {\"}} [HTTP::request]]`

## References ##

http://docs.graylog.org/en/2.0/pages/gelf.html
