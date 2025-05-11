# Purpose

Info on how to read the link records in an Insteon device.

# Example

0 : 0FF8 : A2 00 12.9C.A6 FF 1F 00

FF8 - address in link database

A2 - Flag - Responder link record

00 - Group (Scene) number

12.9C.A6 - Insteon address of Controller

FF - On Level - 255 - 100% On Level

1F - Ramp Rate - 0.1 second - fastest ramp rate

00 - Unit number - normally 00 or 01 - for a KPL the button number

# How It Works

The Group Number and ID (that is, the device or 'target' ID) acts as a signature for responders. The Data-1, Data-2 and Data-3 bytes configures the behavior of the responding device. Note that Data-3 is a button number. For multibutton devices, e.g., Keypads, this is saying "When I see the controller signature I will use the Data- and Data-2 settings in my responder record and apply those to the button# I have in the Data-3 field." So this is how a keypad device determines which of it's 6 or 8 buttons to turn on/off in response to a controller's incoming message.

So - IF a device has a responder record in it's ALdb that matches the **Group+ID signature** that a controller has sent out then it will react as per the info in that response record's three data fields.

Also note that the Controller is suppose to have a Controller record in it's ALdb for every intended Responder in the group. So it doesn't have just a single controller record for a group that it just broadcasts out. It does do that; but supposedly the controller wiill then follow that up with a set of individually directed messages for each device it is a controller for in the given group. It does that by using the redundant controller records in it's ALdb.

All the above is for Insteon scenes. Links to PLMs, and perhaps other devices, may be very different than the above description.

# Insteon All Link Record Format

Below is from the KeyPadLink Developer's guide. See document */ISY-Stuff/Insteon_DevGuide_2334-xxx2486xdev-042013-en.pdf*.

Database entries with Record Control Bit 6: 0 = Responder and Group 1 will control the local load.

Linear ALL-Link Database (AL /L) Record Format:

| Field          | Length<br>(bytes) | Description                                                                                                                                                                                                                                                                                                                             |
| -------------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Record Control | 1                 | Record Control Flag Bits:<br>Bit 7: 1 = Record is in use, 0 = Record is available<br>Bit 6: 1 = Controller (Master) of Device ID, 0 = Responder to (Slave of) Device ID<br>Bit 5: Not used<br>Bit 4: Not used<br>Bit 3: Not used<br>Bit 2: Not used<br>Bit 1: 1 = Record has been used before, 0 = ‘High-water Mark’<br>Bit 0: Not used |
| Group          | 1                 | ALL-Link Group Number this Device ID belongs to                                                                                                                                                                                                                                                                                         |
| ID             | 3                 | Device ID (ID2, ID1, ID0 in that order)                                                                                                                                                                                                                                                                                                 |
| Data 1         | 1                 | On level                                                                                                                                                                                                                                                                                                                                |
| Data 2         | 1                 | Ramp Rate (0x00 -> 0xFE)                                                                                                                                                                                                                                                                                                                |
| Data 3         | 1                 | Button Number (0x01->0x08)                                                                                                                                                                                                                                                                                                              |

To add a record to an AL /L, you search for an existing record that is marked available. (Available means the same as empty, unused or deleted.) If none is available, you create a new record at the end of the AL /L. An unused record will have bit 7 of the Record Control byte set to zero. The last record in an AL /L will have bit 1 of the Record control byte set to zero.

3.1.4 Overwriting an Empty AL /L Record

If you found an empty record, you simply overwrite it with your new record data. Change bit 7 of the Record Control byte from zero to one to show that the record is
now in use. Set bit 6 of the Record Control byte to one if the device containing the AL /L is an INSTEON Controller of the INSTEON Responder Device whose ID is in the record. If instead the device containing the AL /L is an INSTEON Responder to the INSTEON Controller Device whose ID is in the record, then clear bit 6 of the Record Control byte to zero. In other words, within an AL /L, setting bit 6 means “I’m a Controller,” and clearing bit 6 means “I’m a Responder.” Put the ALL-Link Group number in the Group field, and put the Device ID in the ID field. Finally, set the Data 1, Data 2, and Data 3 fields appropriately for the Record Class you are storing.

3.1.5 Creating a New AL /L Record

To create a new record at the end of the AL /T, find the record with bit 1 of the Record Control byte set to zero, indicating that it is the last record in the AL /L. Flip that bit to one.
