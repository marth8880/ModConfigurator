# Configuration Utility

This is a tool that enables Star Wars Battlefront II mod authors to allow players to configure their mods.

Mod authors can set up tabs and dropdown settings through a simple JSON file, cleverly called `config.json`. When a player changes a setting, the settings are saved and munged into a single `.SCRIPT` file as a global Lua table. The settings are simple key(string)-value(number) pairs in the table.

## Config.JSON

The tabs and settings are configured through the bundled `config.json` file. Please see [HERE](https://cheatography.com/gaston/cheat-sheets/json/ "JSON cheat sheet") for more about JSON syntax (but really it's pretty simple).

Under the first pair of brackets, here are the following fields:

| Field | Type | Required | Comment |  
|---|---|---|---|  
| fileVersion | Integer | Yes |  | 
| mungedScriptFileName | String | Yes | File name of the `.SCRIPT` file that gets munged |  
| userConfigLuaTableName | String | Yes | Name of the Lua table that gets compiled into the `.SCRIPT` file |  
| configTabs | Array (objects) | Yes | Collection of tab descriptors |  

An object in the `configTabs` array can have the following fields:

| Field | Type | Required | Comment |  
|---|---|---|---|  
| name | String | Yes | Text name of the tab as displayed in the tab bar |  
| description | String | No | Text description that gets displayed at the top of the tab page |  
| footnote | String | No | Text description that gets displayed at the bottom of the tab page |  
| flags | Array (objects) | Yes | Collection of setting descriptors |  

An object in the `flags` array can have the following fields:

| Field | Type | Required | Comment |  
|---|---|---|---|  
| path | String | Yes | Unique name (a-Z characters only) of the Lua table element that this setting's value gets stored in |  
| name | String | Yes | Text name of the setting that gets displayed next to the setting's dropdown box |  
| toolTipCaption | String | No | Text description of the setting that gets displayed in a tooltip bubble when hovering over the setting's dropdown or name label |  
| values | Array (strings) | Yes | Text names of the setting's possible values, which are shown in the dropdown menu box |  
| defaultValue | Integer | Yes | Default value for the setting, which corresponds with the ordering of the `values` array elements - for example, in an array of `["Apple, "Banana", "Orange"]`, a `defaultValue` of 2 would make the default setting `Orange` |  

## Examples

Consider the following JSON config file:

	{
	  "fileVersion": 1,
	  "mungedScriptFileName": "modconfig",
	  "userConfigLuaTableName": "gModConfig",
	  "configTabs": [
		{
		  "name": "General",
		  "description": "This page contains general settings that affect the mod && its appearance.",
		  "footnote": "Settings marked with * only affect offline && single-player matches.",
		  "flags": [
			{
			  "path": "cfg_MEUnificationEnabled",
			  "name": "Toggle Mass Effect: Unification",
			  "toolTipCaption": "Whether or not the mod's missions should be added to the game.",
			  "values": [
				"Disabled",
				"Enabled"
			  ],
			  "defaultValue": 1
			},
			{
			  "path": "cfg_CustomHUD",
			  "name": "Custom HUD",
			  "values": [
				"Disabled",
				"Enabled"
			  ],
			  "defaultValue": 1
			}
		  ]
		},
		{
		  "name": "Gameplay",
		  "description": "This page contains settings that affect the mod's gameplay.",
		  "footnote": "Settings marked with * only affect offline && single-player matches.",
		  "flags": [
			{
			  "path": "cfg_Difficulty",
			  "name": "* Difficulty",
			  "values": [
				"Casual",
				"Normal",
				"Veteran",
				"Hardcore",
				"Insanity"
			  ],
			  "defaultValue": 1
			},
			{
			  "path": "cfg_AIHeroes",
			  "name": "* AI Heroes",
			  "values": [
				"Disabled",
				"Enabled"
			  ],
			  "defaultValue": 1
			}
		  ]
		}
	  ]
	}

This would generate the following tabs and dropdown settings:

![Generated settings example](Images/generated-settings-1.png)

![Generated settings example](Images/generated-settings-2.png)

When the player changes a setting value in the app, the value is saved as an integer based on the ordering of the settings in the dropdown, starting at 0.

Example of a Lua script that is generated and munged from a player's config (based on the previous JSON example):

    gModConfig = {
		cfg_MEUnificationEnabled = 1,	-- Enabled
		cfg_CustomHUD = 0,				-- Disabled
		cfg_Difficulty = 1,				-- Normal
		cfg_AIHeroes = 1,				-- Enabled
	}

This would then be munged into a file called `modconfig.script`, which can then be loaded into the game with `ScriptCB_DoFile`. The table's values could then be accessed by `gModConfig.cfg_MEUnificationEnabled` etc.

The file gets re-munged each time "Save Changes" is pressed.

## Remarks

* It is recommended to use a unique Lua table name for mods that would load the munged script into a menu (such as in the addme), since more than one script of the same name cannot be loaded into the game at once. The Lua table name can be set via the `userConfigLuaTableName` field in the JSON file. A way to reasonably guarantee a unique table name would be to simply append the mod's ID to the name, e.g. `gMEUModConfig`.
* Each time you add/remove tabs or flags to the JSON config file, you should increase the `fileVersion` number value. This will reset the internally-saved user config and ensure that invalid/nonexistent settings are being loaded.
* The name of the munged `.SCRIPT` file can be changed via the `mungedScriptFileName` field in the JSON config file.