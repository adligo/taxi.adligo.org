Transfer And eXchange Interface
TAXI: not a proper acronym

Use: To transfer and exchange information after a Web Socket connection;
So TAXI is a WebSocket extension.

Important data {key value before semicolon; then data payload frames} to a TAXI connection;
a) actor the name of the user (in computer) or response (system) of the initiate of a WS data transfer.
c) command a string like data structure to instruct the endpoint what to do with the WS data.
g) the endpoint groups that are expected to receive the WS data.
p) the other data bytes in the payload
s) the nodes latest to oldest which have transfered this message. 
u) the endpoint users that are expected to receive the WS data.



oversimplification 3 deployment diagrams
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
   
OK at this point with web sockets I am generally going with the lowest common denominator which means
Extensions in a reasonable time will probably not be something I would have the hope of getting implemented
in a large percentage of browsers in my lifetime so I am using the data only section of the WS protocol to 
add my layer, which means I am NOT compatible with the per message inflate deflate.

The point of TAXI to provide the destination address[es] where the WS data should go only, this allows
routing of the data and augmentation of the routing information.  

The routing information should be augmented to identify the actor (browser user) which sent the message.
For a person using a browser this would be their user id or anonymous token.
 For a responder which handled a rpc call this would be the responders obsfucated identifier.
 
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
   


