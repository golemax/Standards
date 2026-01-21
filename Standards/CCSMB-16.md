# *CCSMB 16:* Modem Over Radio

*Author: Golemax <@golemax>*

*Version: v1.0.0*

*Last updated: YYYY-MM-DD*

## Use case

This standard only apply for the addon of CC:Tweaked named "Classic Peripherals" by AlexDevs.

Due to `Ender Modem Nerfing`, distance be a problems for wireless transmissions.

A radio feature is added with this addon, than extend the transmissions range to 3072 blocks.

But radio is a differente interface than modems, and use it on existing code require to rewrite a large part if old code.

## Basic theory

> A radio station is an radio peripherical capable of reading radio messages

> A radio tower is a radio station capable or transmiting messages to other radio tower

Like modems, radio towers can have a feature like port named "frequency", but in contrary at modems, radio tower can only open one at a time, making impossible to read multiple frequency at the same time with one radio tower.

An answer to this problem is to encapsulate packet data, input and output port on table, this is the `Simple MOR Packet`.

But even radio have a limited range, so to cover the largest possible surface, relays can be used.

But radio don't choose the destination radio station, every messages are brodcasted to every close enought radio station, like modems, and packets are not protected from echo between radio towers.

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
  channel: 0 < number < 65535, 
  replyChannel: 0 < number < 65535,
  payload: any
}
```

## Relay MOR Packet Building

generate from modem.transmit(channel, replyChannel, payload)
and an an id by ( (last_listened_id or random(0, 2^32)) + random(0, 2^24) ) % 2^32
because paquet have a following number, but still random, that can loop every 2^8
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
  id: 0 < number < 4294967295,
  channel: 0 < number < 65535, 
  replyChannel: 0 < number < 65535,
  payload: any
}
```
