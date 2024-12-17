# Notes on the SMTP

## 3.1: Mail

1. SMTP is a series of commands
2. Always starts with a MAIL command- gives sender identification
    1. MAIL \<sp\> FROM:\<reverse-path\> \<CRLF\>
    2. tells the receiver a new transaction is happening
    3. esets all state tables and buffers
    4. gives the reverse path
    5. if acccepted returns 250 okay reply
    6. reverse-path is a reverse source routing list of hosts and source mailbox- the first host in the path should be the one sending the command
3. RCPT commands- gives receiver information
    1. RCPT \<SP\> TO:\<forward-path\> \<CRLF\>
    2. identifies one recipient
    3. if accepted returns 250 and stores forward path- if unknown returns 550
    4. forward-path is source routing list of hosts and destination mailbox- first host is the host receiving the command
4. DATA command- mail data
    1. DATA \<CRLF\>
    2. if accepted returns 354 intermediate reply and considers all following lines to be the message text- when you send the end of text it sends 250 okay
    3. transparency procedure prevenst thsi from looking weird
    4. mail data includes the memo header items
        1. date, subject, to, cc, from, ect.
    5. only fails if the transaction was incomplete
5. End of mail data indicator- confirms transaction
    1. the end of the mail data is indicated by a line containing only a period

## 3.4: Sending and Mailing

1. delivery to mailbox is called 'mailing' - delivery to terminal is called 'sending'
2. SMTP combines the two because they're so similar but sending commands aren't included in required minimum implementation
3. commands that support sending options- used instead of mail and inform the receiver-SMTP of the special semantics
4. SEND \<SP\> FROM:\<reverse-path\> \<CRLF\>
    1. if the user isn't active a 450 reply can be returned to a RCPT command
5. SOML \<SP\> FROM: \<reverse-path\> \<CRLF\>
    1. stands for Send Or Mail
    2. requires mail be sent to the terminal if the user is active, else be sent to their mailbox
6. SAML \<SP\> FROM: \<reverse-path\> \<CRLF\>
    1. Send And Mail

## 3.5: Opening and Closing

1. when the transmission channel opens we ensure the hosts are communicating correctly
2. HELO \<SP\> \<domain\> \<CRLF\>
3. QUIT \<CRLF\>

## 4.1: Commands

1. commands define the system functions
2. all char strings terminated by \<CRLF\>
3. command codes are terminated by \<SP\> if parameters follow, else CRLF
4. syntax of mailboxes has to confirm to receiver site conventions
5. each command has an argument- like the reverse-path, forward-path and mail data
6. there's a buffer for each of these and it's held open unti the transaction is finished
7. MAIL
    1. list of hosts is optional- it indicates the mail was sent through each host- uses names of hosts known in the IPCE where it's relaying the email
    2. the point of this list is to send non-delivery notices to the sender
    3. it's still mandatory because you need it to initiate
8. RCPT
    1. forward-path gives you a set of relays
    2. at every step in the relay the relay host removes itself from the beginning path and puts itself at the beginning of the reverse path
9. DATA
    1. once the EOS character is hit hte receiver processes the reverse-path buffer the forward-path buffer, and the mail data buffer
    2. mailboxes in the return path can be different from sender's mailbox- if you wanted error messages to be put in a different box
10. RSET
    1. aborts the current set of commands and resets all the buffers
11. VERIFY
    1. returns username, full name of user, fully specified mailbox
12. command codes don't have to be capitalized
13. square brackets- optional field
14. the syntax of valid commands in BNF

            <reverse-path> ::= <path>

            <forward-path> ::= <path>

            <path> ::= "<" [ <a-d-l> ":" ] <mailbox> ">"

            <a-d-l> ::= <at-domain> | <at-domain> "," <a-d-l>

            <at-domain> ::= "@" <domain>

            <domain> ::=  <element> | <element> "." <domain>

            <element> ::= <name> | "#" <number> | "[" <dotnum> "]"

            <mailbox> ::= <local-part> "@" <domain>

            <local-part> ::= <dot-string> | <quoted-string>

            <name> ::= <a> <ldh-str> <let-dig>

            <ldh-str> ::= <let-dig-hyp> | <let-dig-hyp> <ldh-str>

            <let-dig> ::= <a> | <d>

            <let-dig-hyp> ::= <a> | <d> | "-"

            <dot-string> ::= <string> | <string> "." <dot-string>

            <string> ::= <char> | <char> <string>

            <quoted-string> ::=  """ <qtext> """

            <qtext> ::=  "\" <x> | "\" <x> <qtext> | <q> | <q> <qtext>

            <char> ::= <c> | "\" <x>

            <dotnum> ::= <snum> "." <snum> "." <snum> "." <snum>

            <number> ::= <d> | <d> <number>

            <CRLF> ::= <CR> <LF>
15. backslash is a quote character here
16. if the host isn't known it can be defined using a code like "[123.255.37.2]" - which indicates an ARPA internet address
17. full definition of the stamp line and the return path line

        <return-path-line> ::= "Return-Path:" <SP><reverse-path><CRLF>
         <time-stamp-line> ::= "Received:" <SP> <stamp> <CRLF>
         <stamp> ::= <from-domain> <by-domain> <opt-info> ";"
         <daytime>
         <from-domain> ::= "FROM" <SP> <domain> <SP>
         <by-domain> ::= "BY" <SP> <domain> <SP>
         <opt-info> ::= [<via>] [<with>] [<id>] [<for>]
         <via> ::= "VIA" <SP> <link> <SP>
         <with> ::= "WITH" <SP> <protocol> <SP>
         <id> ::= "ID" <SP> <string> <SP>
         <for> ::= "FOR" <SP> <path> <SP>
         <link> ::= The standard names for links are registered with
         the Network Information Center.
         <protocol> ::= The standard names for protocols are
         registered with the Network Information Center.
         <daytime> ::= <SP> <date> <SP> <time>
         <date> ::= <dd> <SP> <mon> <SP> <yy>
         <time> ::= <hh> ":" <mm> ":" <ss> <SP> <zone>
         <dd> ::= the one or two decimal integer day of the month in
         the range 1 to 31.
         <mon> ::= "JAN" | "FEB" | "MAR" | "APR" | "MAY" | "JUN" |
         "JUL" | "AUG" | "SEP" | "OCT" | "NOV" | "DEC"
         <yy> ::= the two decimal integer year of the century in the
         range 00 to 99.
        <hh> ::= the two decimal integer hour of the day in the
         range 00 to 24.
         <mm> ::= the two decimal integer minute of the hour in the
         range 00 to 59.
         <ss> ::= the two decimal integer second of the minute in the
         range 00 to 59.
         <zone> ::= "UT" for Universal Time (the default) or other
         time zone designator (as in [2])

## 4.5: Minimum Implementation

1. minimum commands you need to implement
    1. HELO, MAIL, RCPT, DATA, RSET, NOOP, QUIT
2. some objects have minimum and maximum size requirements
    1. user- 64
    2. domain- 64
    3. path- 256
    4. command line- 512
    5. reply line- 512
    6. text line- 1000
    7. recipients buffer- 100

## Appendix E: Reply Codes

1. reply codes theory
2. whole section- just reference this later

Theory of Reply Codes

      The three digits of the reply each have a special significance.
      The first digit denotes whether the response is good, bad or
      incomplete.  An unsophisticated sender-SMTP will be able to
      determine its next action (proceed as planned, redo, retrench,
      etc.) by simply examining this first digit.  A sender-SMTP that
      wants to know approximately what kind of error occurred (e.g.,
      mail system error, command syntax error) may examine the second
      digit, reserving the third digit for the finest gradation of
      information.

         There are five values for the first digit of the reply code:

            1yz   Positive Preliminary reply

               The command has been accepted, but the requested action
               is being held in abeyance, pending confirmation of the
               information in this reply.  The sender-SMTP should send
               another command specifying whether to continue or abort
               the action.

                  [Note: SMTP does not have any commands that allow this
                  type of reply, and so does not have the continue or
                  abort commands.]

            2yz   Positive Completion reply

               The requested action has been successfully completed.  A
               new request may be initiated.

            3yz   Positive Intermediate reply

               The command has been accepted, but the requested action
               is being held in abeyance, pending receipt of further
               information.  The sender-SMTP should send another command
               specifying this information.  This reply is used in
               command sequence groups.

            4yz   Transient Negative Completion reply

               The command was not accepted and the requested action did
               not occur.  However, the error condition is temporary and
               the action may be requested again.  The sender should



               return to the beginning of the command sequence (if any).
               It is difficult to assign a meaning to "transient" when
               two different sites (receiver- and sender- SMTPs) must
               agree on the interpretation.  Each reply in this category
               might have a different time value, but the sender-SMTP is
               encouraged to try again.  A rule of thumb to determine if
               a reply fits into the 4yz or the 5yz category (see below)
               is that replies are 4yz if they can be repeated without
               any change in command form or in properties of the sender
               or receiver.  (E.g., the command is repeated identically
               and the receiver does not put up a new implementation.)

            5yz   Permanent Negative Completion reply

               The command was not accepted and the requested action did
               not occur.  The sender-SMTP is discouraged from repeating
               the exact request (in the same sequence).  Even some
               "permanent" error conditions can be corrected, so the
               human user may want to direct the sender-SMTP to
               reinitiate the command sequence by direct action at some
               point in the future (e.g., after the spelling has been
               changed, or the user has altered the account status).

         The second digit encodes responses in specific categories:

            x0z   Syntax -- These replies refer to syntax errors,
                  syntactically correct commands that don't fit any
                  functional category, and unimplemented or superfluous
                  commands.

            x1z   Information --  These are replies to requests for
                  information, such as status or help.

            x2z   Connections -- These are replies referring to the
                  transmission channel.

            x3z   Unspecified as yet.

            x4z   Unspecified as yet.

            x5z   Mail system -- These replies indicate the status of
                  the receiver mail system vis-a-vis the requested
                  transfer or other mail system action.

         The third digit gives a finer gradation of meaning in each
         category specified by the second digit.  The list of replies

      illustrates this.  Each reply text is recommended rather than
         mandatory, and may even change according to the command with
         which it is associated.  On the other hand, the reply codes
         must strictly follow the specifications in this section.
         Receiver implementations should not invent new codes for
         slightly different situations from the ones described here, but
         rather adapt codes already defined.

         For example, a command such as NOOP whose successful execution
         does not offer the sender-SMTP any new information will return
         a 250 reply.  The response is 502 when the command requests an
         unimplemented non-site-specific action.  A refinement of that
         is the 504 reply for a command that is implemented, but that
         requests an unimplemented parameter.

      The reply text may be longer than a single line; in these cases
      the complete text must be marked so the sender-SMTP knows when it
      can stop reading the reply.  This requires a special format to
      indicate a multiple line reply.

         The format for multiline replies requires that every line,
         except the last, begin with the reply code, followed
         immediately by a hyphen, "-" (also known as minus), followed by
         text.  The last line will begin with the reply code, followed
         immediately by <SP>, optionally some text, and <CRLF>.

            For example:
                                123-First line
                                123-Second line
                                123-234 text beginning with numbers
                                123 The last line

         In many cases the sender-SMTP then simply needs to search for
         the reply code followed by <SP> at the beginning of a line, and
         ignore all preceding lines.  In a few cases, there is important
         data for the sender in the reply "text".  The sender will know
         these cases from the current context.
