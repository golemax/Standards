# *CCSMB 16:* Modem Over Radio

*Author: Golemax <@golemax>*

*Version: v1.0.0*

*Last updated: YYYY-MM-DD*

## Use case

This standard only apply for the addon of CC:Tweaked named "Classic Peripherals" by AlexDevs.

Due to `Ender Modem Nerfing`, distance become a problems for wireless transmissions.

A radio feature is added with this addon, than extend the transmissions range to 3072 blocks.\
But radio is a differente interface than modems, and use it on existing code require to rewrite a large part if old code.

## Basic theory

> A radio station is an radio peripherical capable of reading radio messages

> A radio tower is a radio station capable or transmiting messages to other radio tower

Like modems, radio towers can have a feature like port named "frequency", but in contrary at modems, radio tower can only open one at a time, making impossible to read multiple frequency at the same time with one radio tower.\
An answer to this problem is to encapsulate packet data, input and output port on table, this is the `Simple MOR Packet`.

But even radio have a limited range, so to cover the largest possible surface, relays can be used.\
Also, radio don't choose the destination radio station, every messages are brodcasted to every close enought radio station, like modems, and packets are not protected from echo between radio towers.\
An answer is to identify every paquets by an identifier, and store this identifier for an arbitrary time, this is the `Relay MOR Packet`.

## Simple MOR Packet Building

generate from modem.transmit(channel, replyChannel, payload)
transmit it on frequency 1000

## Simple MOR Packet Listening

listen frequency 1000
parse paquet and resend it on a modem
or read it from a virtual modem by overriding peripherical api and register os event

## Simple MOR Paquet Structure

```lua
{
  channel: 0 <= number <= 65535, 
  replyChannel: 0 <= number <= 65535,
  payload: any
}
```

## Relay MOR Packet Building

generate from modem.transmit(channel, replyChannel, payload)
note: temporary description here for random mean random(a, b) is a fonction generating a random number between where a is included and b is excluded
and an an id by ( (last_listened_id + random(0, 2^24) ) % 2^32) or random(0, 2^32)
it's due to the fact (2^24 * 2^8) = 2^32
so for max identifier increment (2^24) for the random génération, it will take 2^8, so 256 iterations before continue to 0
this exist for preventing collision while random generation based on last paquet id instead or pure random
so, identifiers become incremental for a cycle or 2^8 steps minimum, with a 2^24 number preventing double identifier
Before creating packets, if the program sending the packets is also a listener, it is mandatory to use the identifier of the last packet read
If the listener has not yet read a packet, it is preferable but not mandatory to wait 1 second to receive at least 1 packet
if a packet is received, it is not necessary to wait until the end of the second
if after the second, still no packet is received, the full random generation can be used instead
if the packet sender is not a listener, it can ask an external listener
If no active listener is available, the package creator should not use any registered identifier
transmit it on frequency 1000

## Relay MOR Packet Listening

listen frequency 1000
parse paquet
if id already on list, drop paquet
else, put it in a list for 3 seconds 
possible to drop oldest id if list take 65536 number or more
andd resend it on a modem
or read it from a virtual modem by overriding peripherical api and register os event

## Relay MOR Paquet Structure

```lua
{
  id: 0 <= number <= 4294967295,
  channel: 0 <= number <= 65535, 
  replyChannel: 0 <= number <= 65535,
  payload: any
}
```
