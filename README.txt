Transfer And eXchange Interface
TAXI: not a proper acronym, but chosen because it sounds like what this protocol is trying to accomplish.

Use: To transfer and exchange information transmitted over a Web Socket connection to another Web Socket;
So TAXI is a WebSocket extension.

Important data {key value before semicolon; then other data in payload frames} to a TAXI connection;
a) actor the name of the user (in computer) or response (system) of the initiate of a WS data transfer.
c) command a string like data structure to instruct the endpoint what to do with the WS data.
g) A list of the endpoint groups that are expected to receive the WS data.
   Note the character '*' represents all groups. 
p) the total number of data bytes in the payload including the TAXI header bytes.
s) the nodes latest to oldest which have transfered this message. 
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
   
TAXI uses the the lowest common denominator to pass it's header information which means
Extensions in a reasonable time will probably not be something I would have the hope of getting implemented
in a large percentage of browsers in my lifetime so I am using the data only section of the WS protocol to 
add my layer, which means I am NOT compatible with the per message inflate deflate.  The TAXI header
may be added as a extension later, as a optimization when it is implemented.

The point of TAXI to provide the destination address[es] where the WS data should go only, this allows
routing of the data and augmentation of the routing information.  

The routing information should be augmented to identify the actor (browser user) which sent the message.
For a person using a browser this would be their user id or anonymous token.
 For a responder which handled a rpc call this would be the responders obsfucated identifier 
 (i.e. 'Is1' [for internal server 1]).
 
Examples

Example A;
dd1;
client john sends
   c=text,u=jim,p=156;hey you guys
   serverA augments and sends to to client jim
   a=john,c=text,p=156,s=serverA,u=jim;hey you guys
client jim receives 
   a=john,c=text,p=156,s=serverA,u=jim;hey you guys
   

dd2 (and 3 are the same for TAXI);
client john sends
   c=text,p=156,s=serverA,u=jim;hey you guys
   serverA augments and sends to internalA
   internalA 
   a=john,c=text,p=156,s=serverA,u=jim;hey you guys
internalA stores the message 
   
client jim sends
   c=getTexts,p=0;
   serverB augments and sends to internalC
   a=jim,c=getTexts,p=0;
   internalC gets the message and responds with
   a=internalC,c=passTexts,p=256;texts;
   <text id="1" from="jim">text abcedfg test</text>
   <text id="2" from="jim">hey you guys</text>
client jim displays tests and sends
   c=deleteTexts,p=7;ids=1,2
   
The TAXI header is simply a key value comma separated list of items,
for keys that may have multiple items the curly braces are used; 
I.E.;
a=scott,g={devs,admins},c=text

TAXI headers are white space and case sensitive, white space can 
be added using curly braces;
a=scott,g={ admins },c=text
the above would text the ' admins ' group not the 'admins' group.

