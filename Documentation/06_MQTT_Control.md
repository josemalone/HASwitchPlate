# MQTT Control

## MQTT Namespace
By default the device will subscribe to `homeassistant/haswitchplate/<node_name>/#` to accept incoming commands.  There are two subtopics to send commands to and receive messages from the panel:

* `command` will send commands or set attribute of the display, such as button text or screen dim.  The specific attribute must be appended as a subtopic, with the value to be set delivered as the payload.  For example, the following command will set the text displayed on page 1/button 4 to "Lamp On": `mosquitto_pub -h mqtt -t homeassistant/haswitchplate/nodename/command/p[1].b[4].txt -m '"Lamp On"'`
* `state` topics will be sent by the panel in response to local user interactions or received commands which provide output in return.  For example, a user pressing button 4 on page 1 will cause the panel to publish a message: `'homeassistant/haswitchplate/nodename/command/p[1].b[4]' 'ON'`

## `command` Syntax
Messages sent to the panel under the `command` topic will be handled based on the following rules:
* **`'[...]/command' 'dim 100'`** A `command` with no subtopic will send the command in the payload to the panel directly.
* **`'[...]/command/page' '1'`** The `page` command subtopic will set the current page on the device to the page number included in the payload.
* **`'[...]/command/update' 'http://192.168.0.10:8123/local/HASwitchPlate.tft'`** The `update` command subtopic attempts to update the Nextion LCD from the file URL named in the payload.
* **`'[...]/command/p[1].b[4].txt' '"Lamp On"'`** A `command` with a subtopic will set the attribute named in the subtopic to the value sent in the payload.  You can send these messages with `retain` enabled and the broker will remember these settings for you.
* **`'[...]/command/p[1].b[4].txt' ''`** A `command` with a subtopic and an empty payload will request the current value of the attribute named in the subtopic from the panel.  The value will be returned under the `state` topic as `'[...]/state/p[1].b[4].txt' '"Lamp On"'`

[Detailed documentation on the Nextion command instruction set can be found here](https://www.itead.cc/wiki/Nextion_Instruction_Set).  Note that some values require quotes, while others don't, and different shells (and Home Assistant config files) eat quotes differently.  As a general rule, attributes which accept an numeric value cannot have quotes, all other attributes must be enclosed in double quotes.

Objects are referenced by page number (not page name) and object ID (not object name) as shown in the Nextion editor.  The Nextion notation for each object is of the form `p[1]b[2]` meaning page number 1, object number 2.  Confusingly, button objects will be named starting at b0 but it might be object 2, or 7, etc. Other objects will be named similarly with different letters.  For example, the first text field on the page will be automatically named t0.  These names have nothing to do with the object ID, and all objects regardless of type are still referenced as `p[<page number>].b[<object id>]`

![Page and Object IDs](https://github.com/aderusha/HASwitchPlate/blob/master/Documentation/Images/Nextion_Editor_Page_and_Object_Ids.png?raw=true)

While it is possible to send `command` messages using the object or page names, all messages coming back from the panel under `state` will be using page numbers and object IDs.  Save yourself the confusion and do everything in the format `p[<page number>].b[<object id>]` and things will go easier.
