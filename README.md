# Attacking laptops that are protected by Microsoft Bitlocker drive encryption

The background for this came from the recent blog post https://dolosgroup.io/blog/2021/7/9/from-stolen-laptop-to-inside-the-company-network and from the earlier https://labs.f-secure.com/blog/sniff-there-leaks-my-bitlocker-key/ post, they’re well worth a read.


## This is what we have.

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899051919140589578/FA2-2PVWQAISyj1.jpeg?width=885&height=664)

  
## PC specs

- It’s a Dell E7450 with Windows 10 Professional and BitLocker full drive encryption enabled. Keys are stored in the onboard TPM 1.2 module

  
We have physical access to a turned off laptop in a hotel room. When I do this I first like to do a full tear down of the machine. Googling for ‘motherboard replacement’ is usually a good start and for me, up pops this video.

To open the notebook, there is a video. 

```bash
  https://youtu.be/jfP7EamGw98
```

This is what we’re left with.

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899053356511797248/FA2-34RXoAAZ-5v.png?width=885&height=664)

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899053382831054848/FA2-35RXoAMSAnv.png?width=885&height=664)

## Where is it ?

We need to get access to the TPM module for this attack, but we have no idea what it looks like.

To fix this we’re going to start picking chips at random and Googling the writing on top of them. You end up with diagrams that look a bit like this

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899094947695824896/FA2-72VXIAM2PS_.png?width=1142&height=664)

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899094948035563520/FA2-70GXsAIxwr9.png?width=1072&height=664)

Bingo! We’re found the TPM module. 

A google for the data sheet turns up http://ww1.microchip.com/downloads/en/devicedoc/Atmel-8884S-TPM-AT97SC3205-Datasheet-Summary.pdf which tells us it communicates over the SPI protocol and is in a TSSOP28 chip, which is not exactly easy to connect to. As seen in the Dolos attack there might be another way though

For cost and simplicity, manufacturers often bundle several chips onto the same SPI ‘bus’. If we can find another chip that’s easy to connect to on the same bus, we can sniff the key off the wire.

What I’m doing here is ‘walking’ my multimeter over the TPM pins while connected to various different chips to see if any of them ‘beep’ to tell us there is a connection.

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899095847076266055/FA2-_HOWUAUfpCS.png?width=645&height=664)

Unfortunately, no dice. Either that means there isn’t anything else on that SPI bus, or it might mean there is a resistor in the way. 

A google for ‘e7450 schematic’ turns up this diagram though which looks promising.

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899096877738369044/FA2_BESWQAEQBC6.png?width=943&height=664)

Look back earlier where I labelled the chips. There are two Winbond flash modules that look like they might be the ones on the diagram, so this attack might not be dead in the water yet. Trouble is, our laptop won’t do anything when it’s in bits…

Reassembling the laptop again and we can see those little flash chips just poking out. All we had to do to access them in this case was remove the back covers, which is 8 screws.

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899097805447114782/FA2_B0JXsAI-Gt5.png?width=497&height=664)

To determine whether is an attack vector or not, we’re going to have to hook up a logic analyser. We’re using  a Salae here because there is a nice extension for it that’ll pull out any bitlocker keys it sees. We just connect up and boot the laptop.

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899098237741445141/FA2_DWRXsAAaD2h.png?width=877&height=664)

Success! With the laptop booting come booting into Windows, the analyser has sniffed all of the SPI communications and the script has found a bitlocker key hiding in there. Remember to invert the ‘enable’ line if you’re doing this yourself and capture at a good speed.

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899098497951883274/FA2_FVtXIAEhTSH.png?width=1440&height=476)

Now you’ve got a key, we need a way to decrypt the hard drive. Here, I’ve pulled the drive out but you could do it in the machine if it boots from USB. As you can see, Kali sees an encrypted drive.

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899099008771948575/FA2_F_2WEAQ2RCA.png?width=498&height=664)

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899098887283957770/FA2_F-2WQAA6aKm.png?width=1000&height=664)

As we have the BitLocker VMK key, we can now use the Dislocker toolkit in Kali to decrypt it.

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899102823713763378/FA2_IS4WEAk8PVZ.png?width=924&height=664)

But stealing data is one thing, all we have access to is what’s contained on the laptops drive. What makes this attack so insidious is that we can also plant malware on the drive to execute on next boot, giving us access to all of the CEOs data and a foot inside the organisation

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899099386724900924/FA2_JIMWQAAULwK.png)

![App Screenshot](https://media.discordapp.net/attachments/842488667653275648/899099555134603284/FA2_JJNXoAgynVh.png?width=885&height=664)

With a little prior knowledge, this entire attack can be pulled off in as little as 10 mins, it’s certainly a viable attack method.
