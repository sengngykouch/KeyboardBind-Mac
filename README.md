# Poker3 Keyboard Layout for Mac


## Description
- We use both [hammerspoon](http://www.hammerspoon.org/) and [karabiner-elements](https://karabiner-elements.pqrs.org/) to create custom key bindings.
- This preset allows you to keep your hands on home rows all the time. Hence it increases your productivity.
- This works by using Caplock as the modifier key, then we bind it with other keys to trigger many most used keys such as Arrow, Delete, Backspace keys, etc.

## Operating System Requirements
- MacOS Only. For Window version, please visit [here](https://github.com/sengngykouch/keyboardBind) for more info.

## Installation and Setup
- Download and install both of these: 
   - [karabiner-elements](https://karabiner-elements.pqrs.org/)
   - [hammerspoon](http://www.hammerspoon.org/)

#### 1. Set up Karabiner Elements
   1. Open Preference from [Karabiner Elements](https://karabiner-elements.pqrs.org/) app on the menu bar.
   2. Choose **Simple modifications** tab.
   3. Choose **For all devices** from Target device dropdown.
   4. Click **Add item**.
   5. Choose **caps_lock** for From Key.
   6. Choose **fn** for To Key.
   7. These will map the **caps-lock** to the **fn** key.
   
   ![](https://github.com/sengngykouch/KeyboardBind-Mac/blob/main/ReadMe-Support-Assets/Karabiner-Elements-setup.gif)
   
#### 2. Set up Hammerspoon
   1. Choose **Open Config** from the [Hammerspoon](http://www.hammerspoon.org/) app on the menu bar
   2. Copy and replace all the content with [init.lua](https://github.com/sengngykouch/KeyboardBind-Mac/blob/main/init.lua) file. Or from below here:
      ```lua
      local module = {}

      module.debugging = false -- whether to print status updates

      local eventtap = require "hs.eventtap"
      local event    = eventtap.event
      local inspect  = require "hs.inspect"

      local keyHandler = function(e)
      -- Add more key bind down here.
      local watchFor = {
               j = "left",
               k = "down",
               i = "up",
               l = "right",
               space = "delete",
               f = "forwarddelete",
               h = "home",
               n = "end"            
           }
      local actualKey = e:getCharacters(true)
      local replacement = watchFor[actualKey:lower()]

       if replacement then
           local isDown = e:getType() == event.types.keyDown
           local flags  = {}

           for k, v in pairs(e:getFlags()) do
               if v and k ~= "fn" then -- fn will be down because that's our "wrapper", so ignore it
                   table.insert(flags, k)
               end
           end

           if module.debugging then print("viKeys: " .. replacement, inspect(flags), isDown) end

           local replacementEvent = event.newKeyEvent(flags, replacement, isDown)
            if isDown then
               -- allow for auto-repeat
               replacementEvent:setProperty(event.properties.keyboardEventAutorepeat, e:getProperty(event.properties.keyboardEventAutorepeat))
            end

            return true, { replacementEvent }
         else
            return false -- do nothing to the event, just pass it along
         end
      end

      local modifierHandler = function(e)
          local flags = e:getFlags()
          local onlyControlPressed = false

          for k, v in pairs(flags) do
              onlyControlPressed = v and k == "fn"
              if not onlyControlPressed then break end
          end

          -- you must tap and hold fn by itself to turn this on
          if onlyControlPressed and not module.keyListener then
              if module.debugging then print("viKeys: keyhandler on") end
              module.keyListener = eventtap.new({ event.types.keyDown, event.types.keyUp }, keyHandler):start()
          -- however, adding additional modifiers afterwards is ok... its only when fn isn't down that we switch back off
          elseif not flags.fn and module.keyListener then
              if module.debugging then print("viKeys: keyhandler off") end
              module.keyListener:stop()
              module.keyListener = nil
          end

          return false
      end

      module.modifierListener = eventtap.new({ event.types.flagsChanged }, modifierHandler)

      module.start = function()
          module.modifierListener:start()
      end

      module.stop = function()
          if module.keyListener then
              module.keyListener:stop()
              module.keyListener = nil
          end
          module.modifierListener:stop()
      end

      module.start() -- autostart

      return module
      ```
   3. Save.
   4. Then **Reload Config**.
   
## Usage
![alt text](https://github.com/sengngykouch/keyboardBind/blob/master/vortex-poker3-62uk_zpskowcfdjx.jpg "Key binds")  

1. Keyboard Hotkeys:  
  - Hold down the *Capslock* (Blue color) + *another Key* (Green color).  
    - Eg: `Hold down *Capslock* and Press *j* will trigger *Left-Arrow*.`
    - Eg: `Hold down *Capslock* and Press *n* will trigger *End*.`
    - Eg: `Hold down *Capslock* and Press *h* will trigger *Home*.`



**Note: These keys can bind to more key.**  
- Eg: `Caps + Shift + j` =  select while moving to the left.
   
