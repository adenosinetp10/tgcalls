## Examples

All examples are licensed under the [CC0 License](LICENSE) and are therefore fully 
dedicated to the public domain. You can use them as the base for your own bots 
without worrying about copyrights.

### [player_as_smart_plugin.py](player_as_smart_plugin.py)

An example of a smart plugin for Pyrogram. Plays a music by replaying
to contained audio message via the command `!play`. Has the following commands:
`!join`, `!leave`, `!rejoin`, `!replay`, `!stop`, `!volume`, `!mute`, `!unmute`, `!pause`, `!resume`.
Reacts only to commands from the owner.

### [radio_as_smart_plugin.py](radio_as_smart_plugin.py)

Example of using stream url in the set of voice chats (many VC at the same time).
Reacts only to commands from anonymous admins.

### [recorder_as_smart_plugin.py](recorder_as_smart_plugin.py)

Pyrogram Smart Plugin for recording a voice chat for 30 seconds and send recorded opus file along with
audio info to the group chat, just one owner only command `!record`.

### [playout.py](playout.py)

An example of joining a voice chat and playing music. 
Description of all GroupCall methods. 
Temporarily use instead of documentation.
