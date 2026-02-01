# Mission Pinball Framework - Claude Code Agents Setup

✅ **Successfully configured 9 specialized Claude Code agents for MPF development!**

## 📋 What Was Created

### Agent Files (in `.claude/agents/`)

1. **mpf-config-tester.md** - ⭐ Config testing and validation expert
2. **mpf-config-expert.md** - Hardware configuration expert
3. **mpf-mode-builder.md** - Game mode creation specialist
4. **mpf-show-creator.md** - LED show and animation designer
5. **mpf-hardware-debugger.md** - Hardware troubleshooting expert
6. **mpf-scoring-logic.md** - Scoring and player variable specialist
7. **mpf-multiball-wizard.md** - Multiball and wizard mode expert
8. **mpf-display-designer.md** - Display and widget designer
9. **mpf-sound-designer.md** - Audio system specialist

### Documentation

- **README.md** in `.claude/agents/` with full usage guide
- **This file** as a quick setup confirmation

---

## ⭐ Featured: mpf-config-tester

**The most important agent for your workflow!**

After making any config changes, use this agent to:
- Run `mpf both -X` in smart virtual mode
- Analyze logs for errors and warnings
- Understand that **NO ERRORS ≠ PERFECT**
- Get a specific hardware test plan

**Example:**
```
"Test my new multiball config and tell me what needs hardware testing"
```

The agent will:
1. ✅ Run virtual test
2. 📋 Report any errors/warnings
3. 🧪 Create hardware test plan
4. 💡 Suggest improvements

---

## 🚀 How to Use These Agents

### Method 1: Natural Language (Automatic Selection)
Claude Code will automatically select the right agent based on your request:

```
"Add 5 new RGB LEDs to my playfield"
→ Automatically uses mpf-config-expert

"Create a multiball mode with virtual locks"
→ Automatically uses mpf-multiball-wizard

"Make a rainbow chase effect"
→ Automatically uses mpf-show-creator
```

### Method 2: Explicit Agent Call
Use the Task tool to explicitly call an agent:

```
"Use the mpf-mode-builder agent to create a new mansion mode"
```

### Method 3: Direct Reference
Just mention what you want and the system will route to the right expert:

```
"Why isn't my trough ejecting balls? Check the logs."
→ Routes to mpf-hardware-debugger
```

---

## 🎯 Quick Examples

### Example 1: Add New Hardware
```
"Add these switches to my config:
- Left ramp entry (s_left_ramp_entry) at cab-20
- Right ramp entry (s_right_ramp_entry) at cab-21"
```
→ Uses **mpf-config-expert**

### Example 2: Create a Mode
```
"Create a 'graveyard mode' that:
- Tracks 8 target hits
- Awards 10k per target
- Has a 30-second timer
- Plays spooky sounds
- Uses purple lights"
```
→ Uses **mpf-mode-builder**

### Example 3: Design Light Show
```
"Make an attract mode show that cycles through:
rainbow colors on all playfield LEDs,
repeating every 2 seconds"
```
→ Uses **mpf-show-creator**

### Example 4: Implement Scoring
```
"Create a combo system:
- Hitting any ramp starts a 5-second timer
- Each additional ramp adds 1x multiplier
- Max 5x multiplier
- Resets when timer expires"
```
→ Uses **mpf-scoring-logic**

### Example 5: Debug Hardware
```
"My left flipper fires but feels weak.
Check the config and suggest fixes."
```
→ Uses **mpf-hardware-debugger**

---

## 📁 File Structure

```
mpf-pinball/
├── .claude/
│   ├── agents/                    ← Agent definitions
│   │   ├── mpf-config-expert.md
│   │   ├── mpf-mode-builder.md
│   │   ├── mpf-show-creator.md
│   │   ├── mpf-hardware-debugger.md
│   │   ├── mpf-scoring-logic.md
│   │   ├── mpf-multiball-wizard.md
│   │   ├── mpf-display-designer.md
│   │   ├── mpf-sound-designer.md
│   │   └── README.md
│   └── settings.local.json        ← Permissions
├── config/                        ← Your MPF configs
│   ├── config.yaml
│   ├── hardware.yaml
│   ├── switches.yaml
│   ├── coils.yaml
│   └── lights.yaml
├── modes/                         ← Your game modes
├── shows/                         ← Your light shows
├── sounds/                        ← Your audio files
└── MPF_CLAUDE_AGENTS_SETUP.md    ← This file
```

---

## 🔍 Agent Specializations

| Agent | Best For | Example Use |
|-------|----------|-------------|
| **config-tester** ⭐ | Testing/Validation | "Test my config and create test plan" |
| **config-expert** | Hardware setup | "Add 10 LEDs starting at channel 3-0" |
| **mode-builder** | Game modes | "Create a skill shot mode" |
| **show-creator** | Light animations | "Make a red-blue chase effect" |
| **hardware-debugger** | Troubleshooting | "Why won't my coil fire?" |
| **scoring-logic** | Scoring systems | "Build a 5x multiplier system" |
| **multiball-wizard** | Multiballs | "Create 3-ball multiball with jackpots" |
| **display-designer** | Screens/slides | "Design a jackpot celebration slide" |
| **sound-designer** | Audio | "Set up music ducking for callouts" |

---

## 💡 Pro Tips

1. **Be Specific**: "Add left ramp shot at s_left_ramp_entry" > "add a shot"

2. **Provide Context**: Mention which config file or mode you're editing

3. **Reference Existing**: "Like the mansion mode but for the swamp"

4. **Test Incrementally**: One feature at a time, test, then continue

5. **Verify Hardware**: Always check hardware numbers match your setup

6. **Use Verbose Logs**: Run `mpf both -vV` when debugging

---

## 🛠️ Common MPF Commands

```bash
# Run game + media controller
mpf both

# Run with full logging
mpf both -vV

# Run in virtual mode (no hardware needed)
mpf both -X -vV

# Run game only (no display)
mpf game -b -vV

# Check config syntax
mpf both -c config/config.yaml
```

---

## 📚 Resources

- **Agent README**: `.claude/agents/README.md`
- **MPF Docs**: https://missionpinball.org/latest
- **MPF Forum**: https://groups.google.com/g/mpf-users
- **Your MPF Guide**: `mpf-docs/deepseek-mpf.md`

---

## ✅ Verification

Agents are configured and ready! Test with:

```
"List all the agents available for MPF development"
```

or start using them immediately:

```
"Create a new pop bumper mode that lights super pops at 50 hits"
```

---

## 🎉 Next Steps

1. **Try an agent** - Pick one of the examples above
2. **Read the README** - Check `.claude/agents/README.md` for details
3. **Build your game** - Start creating modes and features!
4. **Experiment** - Try combining multiple agents for complex tasks

---

**Happy Pinball Development! 🎮⚡**

*Created: October 2025*
*MPF Version: 0.80+*
*Total Agents: 9*
*Total Agent Definitions: 1,673 lines*
