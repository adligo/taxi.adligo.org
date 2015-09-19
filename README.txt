Transfer And eXchange Interface protocol
Informal Draft 09/18/2015

TAXI: not a proper acronym, but chosen because it sounds like what this protocol is trying to accomplish.
TAXI is a replacement for asdcp.adligo.org, the initial proof of concept. 

Use: To transfer and exchange or route information transmitted over a HTTP(s) (or Web Socket (wss)) connection one or 
more other HTTP(s) connections (Web Socket(s) (wss)).  The point of TAXI to provide the destination address[es] where the WS 
data should go and where it has been and who or what sent it only.  This allows other protocols
to extend this draft and use a variety of data formats (Utf-8 text, Xml, Binary, etc).

TAXI is comprised of a Utf-8 character string/header which exists in the data section of a Web Socket message,
or may also exist in the extension section of a web socket message if supported by the client and server.
For regular HTTP the TAXI header is the first text in the EntityBody section of the message.
Usage of the extension section is preferred since it is faster to parse if the message data
is compressed, however it may not be implemented by various Web Socket clients.  TAXI
should remain compatible with all Http and Web Socket clients, including those that never even had 
a protocol version number.
   The TAXI header consists of list of simple key=value pairs separated by commas and ending in a semicolon.
I.E.;
a=scott,g={devs,admins},c=text

TAXI headers are white space and case sensitive, white space can 
be added using curly braces, other wise white space should be removed;
a=scott,g={ admins },c=text
the above would text the ' admins ' group not the 'admins' group.
Values inside of curly braces may also be wrapped by the double quote character '"',
this should be compatible with most LDAP and other Authentication systems.
I.E.;
a=scott,c=text,p=10,t=1,u={"john","pa\"ul"};0987654321

Double quotes in user names can be escaped with the character sequence backslash double quote '\"'.
http://www-12.lotus.com/ldd/doc/domino_notes/7.0/help_admin_upgrade7.nsf/b3266a3c17f9bb7085256b870069c0a9/b741831654e21bb4852570bb0050038f
http://www.openldap.org/lists/openldap-software/200204/msg00563.html

Important TAXI data keys {key value before a semicolon; then other data in payload frames} to a TAXI connection;
a) actor the name of the user (in computer) or handler (system responder) of the initiate of a WS data transfer.
c) command a string like data structure to instruct the endpoint what to do with the WS data.
g) A list of the endpoint groups that are expected to receive the WS data.
   Note the character '*' represents all groups. 
p) the total number of data bytes in the payload including the TAXI header bytes.
s) the nodes latest to oldest which have transfered this message. 
t) The unique identifier of a browser tab, with in the scope of the actor (Person using a Web Browser)
   server combination.  This key should always be present when a Browser sends a TAXI message,
   and may be present to route a message to a particular browser window when a Handler (server or internal server) 
   sends a message.
u) the endpoint users that are expected to receive the WS data.
     Note the character '*' represents all users.

   
In all deployments of TAXI the Server would interpret the command and decide what to 
do with the message.   There are some text command examples below, but the command could 
be anything (i.e. getAccounts, deleteAccount, updatePassword etc).

oversimplification of 3 deployment diagrams in text (also see diagrams/taxi_deployments.*)
dd1)
  3 teir
  browser a -> ws -> server
         send message 1
  browser b <- ws <- server
         receive message 1
dd2 scaled ws pn)
   browser a -> ws -> server a -> store a
      send message 1
      pn 
                      server b -> store a = b
   brower b -> ws -> server b -> store b
      receive message 1
      
dd3 scaled ws vpn)
   browser a -> ws -> server a -> store a
      send message 1
      vpn 
                      server b -> store a = b
   brower b -> ws -> server b -> store b
      receive message 1
   


The TAXI header (routing information) should be augmented to identify the actor (browser user) which sent the message.
For a person using a browser this would be their user id or anonymous token.
 For a handler which responded to a  rpc call this would be the handlers identifier 
 (i.e. 'Is1' [for internal server 1]).
 
Examples

Example A;
dd1;
client john sends
   c=text,t=1,u=jim,p=156;hey you guys
   serverA augments and sends to to client jim
   a=john,c=text,p=156,s=serverA,t=1,u=jim;hey you guys
client jim receives 
   a=john,c=text,p=156,s=serverA,t=1,u=jim;hey you guys
   

dd2 (and 3 are the same for TAXI);
client john sends
   c=text,p=156,s=serverA,t=1,u=jim;hey you guys
   serverA augments and sends to internalA
   internalA 
   a=john,c=text,p=156,s=serverA,t=1,u=jim;hey you guys
internalA stores the message 
   
client jim sends
   c=getTexts,p=0,t=1;
   serverB augments and sends to internalC
   a=jim,c=getTexts,p=0,t=1;
   internalC gets the message and responds with
   a=internalC,c=passTexts,p=256,t=1;texts;
   <text id="1" from="jim">text abcedfg test</text>
   <text id="2" from="jim">hey you guys</text>
client jim displays tests and sends
   c=deleteTexts,p=7,t=1;ids=1,2
   
Note the above text xml nodes are not part of the TAXI specification.
Also note TAXI can not be layered over Bayeux since JSON must be used everywhere in Bayeux.