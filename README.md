

/**
 *  Plex Integration
 *
 *  Copyright 2015 Christian Hjelseth
 *
 *  Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except
 *  in compliance with the License. You may obtain a copy of the License at:
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 *  Unless required by applicable law or agreed to in writing, software distributed under the License is distributed
 *  on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License
 *  for the specific language governing permissions and limitations under the License.
 *
 *  v1.4.1 - Fixed bug.
 *  -------------------------------------
 *  v1.4 - Added ability to execute routines. Fixed modes bug.
 *  -------------------------------------
 *  v1.3 - Added multiple rooms/players support, and config for when trailers play.  
 *  -------------------------------------
 *  v1.2 - Added ability to view api info with the App.  Also added ability to toggle debug logging on and off (@tonesto7)
 *		   Added ability to set certain modes only to trigger switches
 *  -------------------------------------
 *  v1.1 - Added optional mode changes
 */
 
import groovy.json.JsonBuilder

definition(
    name: "Plex Integration",
    namespace: "ChristianH",
    author: "Christian Hjelseth",
    description: "Allows web requests to dim/turn off/on lights when plex is playing.",
    category: "My Apps",
    iconUrl: "http://1sd3vh2v9afo91q38219tlj1.wpengine.netdna-cdn.com/wp-content/uploads/2015/05/plex-icon-server-big-3b6e2330294017827d0354f0c768a3ab.png",
    iconX2Url: "http://1sd3vh2v9afo91q38219tlj1.wpengine.netdna-cdn.com/wp-content/uploads/2015/05/plex-icon-server-big-3b6e2330294017827d0354f0c768a3ab.png",
    iconX3Url: "http://1sd3vh2v9afo91q38219tlj1.wpengine.netdna-cdn.com/wp-content/uploads/2015/05/plex-icon-server-big-3b6e2330294017827d0354f0c768a3ab.png",
    oauth: [displayName: "PlexServer", displayLink: ""])

preferences {
	page(name: "configPage", nextPage: "roomsPage") 
	page(name: "roomsPage")
}

//had to move to a dynamic page to handle the possible missing token
/*  Main Page  */
def configPage() {
	//Generates an accessToken if one has not been generated
	if (!state.accessToken) {
    	createAccessToken()
   	}
    if (state.debugLogging == null) 	{ state.debugLogging = false }
    
	dynamicPage(name: "configPage", title: "Plex2SmartThings", uninstall: true) {
        // Enables logging debug only when enabled
        section(title: "Debug Logging") {
       		paragraph "If you experiencing issues please enable logging to help troubleshoot"
            input "debugLogging", "bool", title: "Debug Logging...", required: false, defaultValue: false, refreshAfterSelection: true
            	
            if (debugLogging) { 
            	state.debugLogging = true 
                logWriter("Debug Logging has been ${state.debugLogging.toString().toUpperCase()}")
                paragraph "Debug Logging is Enabled: ${state.debugLogging}"
            }
            else { 
            	state.debugLogging = false 
            	logWriter("Debug Logging has been ${state.debugLogging.toString().toUpperCase()}")    
            }
    	}
    	section() { 
        	href url: "https://graph-na04-useast2.api.smartthings.com/location/installedSmartApps/2722b96f-d4f2-457d-8272-e803bdeb6ff8}", 
            		style:"embedded", required:false, title:"API Information", description: "Tap to view Info"
    	}
		section("Rooms") {
            paragraph "On the next page you can define up to 5 rooms."
		}
    }
}

/*  Rooms Page  */
def roomsPage() {
	dynamicPage(name: "roomsPage", title: "Rooms", install: true, uninstall: true) {
		section("Info") {
            paragraph "Each room can have up to 2 plex player names that control it. You may set the name to * in order to match any player. Only the first matching room will be used."
		}
		
		//It didn't work to loop through anything but devices. The values didn't save for some reason.. Will be fixed later sometime.
		
		//Get defined routines
		def actions = location.helloHome?.getPhrases()*.label
		if (actions) {
			actions.sort()
		}
		
		//ROOM 1
		section("Rooms #1") {
            input(name: "611eb0138919ce4b-com-plexapp-android", type: "text", title: "Plex player name", required:true)
            input(name: "playerB1", type: "text", title: "Plex player name (alternative)", required:false)
        }
		section("Devices (room #1)") {
			input "hues1", "capability.colorControl", title: "Hue Bulbs", multiple: false, required:false
			input "dimmers1", "capability.switchLevel", title: "Dimmers", multiple: true, required: false
			input "switches1", "capability.switch", title:"Switches", multiple: true, required: false
			input "playMode1", "mode", title: "Mode when playing", required:false
			input "pauseMode1", "mode", title: "Mode when paused", required:false
			input "stopMode1", "mode", title: "Mode when stopped", required:false
			if (actions) {
				input "playRoutine1", "enum", title: "Routine when playing", required:false, options: actions
				input "pauseRoutine1", "enum", title: "Routine when paused", required:false, options: actions
				input "stopRoutine1", "enum", title: "Routine when stopped", required:false, options: actions
			}
		}
		section("Config (room #1)") {
			input(name: "bSwitchOffOnPause1", type: "bool", title: "Turn switches off on pause")
			input(name: "bDimOnlyIfOn1", type: "bool", title: "Dim bulbs only if they're already on")
			input(name: "bTreatTrailersAsPause1", type: "bool", title: "Use pause config for movie trailers")
			input(name: "iLevelOnStop1", type: "number", title: "Bulb/dimmer levels on Stop", defaultValue:100)
			input(name: "iLevelOnPause1", type: "number", title: "Bulb/dimmer levels on Pause", defaultValue:30)
			input(name: "iLevelOnPlay1", type: "number", title: "Bulb/dimmer levels on Play", defaultValue:0)
		}
