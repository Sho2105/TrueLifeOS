### **Full 2-Button Action Table**

| Action Type             | Button A   | Button B   | Notes                      |
| ----------------------- | ---------- | ---------- | -------------------------- |
| **Single**              | Tap        | -          | Simple hit                 |
| Single                  | Hold       | -          | Charge/long action         |
| Single                  | Double Tap | -          | Quick burst                |
| Single                  | -          | Tap        | Mirror of A                |
| Single                  | -          | Hold       | Mirror of A                |
| Single                  | -          | Double Tap | Mirror of A                |
| **Simultaneous**        | Tap        | Tap        | Both buttons at once       |
| Simultaneous            | Hold       | Hold       | Both buttons held together |
| Simultaneous            | Tap        | Hold       | Tap A while holding B      |
| Simultaneous            | Hold       | Tap        | Hold A while tapping B     |
| **Sequential Hold-Tap** | Hold → Tap | -          | Press A hold, then B tap   |
| Sequential Hold-Tap     | Tap → Hold | -          | Press A tap, then B hold   |
| Sequential Hold-Tap     | -          | Hold → Tap | Press B hold, then A tap   |
| Sequential Hold-Tap     | -          | Tap → Hold | Press B tap, then A hold   |
| **Double-tap Combos**   | Double Tap | Tap        | A double tap + B tap       |
| Double-tap Combos       | Tap        | Double Tap | A tap + B double tap       |
| Double-tap Combos       | Double Tap | Double Tap | Both double tap            |
---

1. **Single taps** → essential, instinctual.
2. **Holds** → useful for charging or risk moves, okay if only one button at a time.
3. **Double tap** → works if limited to one button, but timing window must be generous.
4. **Simultaneous tap + hold combos** → keep 1–2 (like tap A + hold B) for “oh crap I need a reactive move” but don’t overcomplicate.
5. **Sequential stuff** → probably skip in a game requiring split-second reactions; your brain can’t reliably hold “hold A then tap B” under stress.
6. **Double-tap + button combos** → skip, too precise, too stressful.

**Comfortable set** 
Speed tier are count dynamically based on current speed (From tier 0 to 5) 0 stop, 1 = base speed, same speed tier clash, higher speed tier damages lower ones
* Tap A = flick short distance, cd 1s (dynamic from 1-4) does not reset speed after use
* Tap B = parry (base parry duration=0.2s, cd 1.0s) stun parried enemy for 1 sec base, higher the enemy's speed tier parried, +0.5 sec stun,
* Hold A = accelerate slowly to x4 base speed (increase speed tier from 1-4 slowly the faster the harder it takes to turn/aim)
* Hold B = decelerate fast to x0.5 base speed (reduce parry cd to 0.5s)
* Double Tap A = item drop use 
* Double Tap B = skill drop use
* Tap A + Tap B = aoe push, 5s cd ()
* Tap A + Hold B = dash foward based on momentum stored and release shocktrail behind (dynamic from 1-4) reset speed to 1 after use
* Hold A + Tap B = sudden stop and release shocktrail foward based on current speed (dynamic from 1-4) reset speed to 1 after use
* Hold A + Hold B = charge infinitely nuke, unparry-able (tier 5) speed drop to 0 during charge.

| Input           | Function                           | Tier Interaction | Window / Duration | CD                      | Notes                                                    |
| --------------- | ---------------------------------- | ---------------- | ----------------- | ----------------------- | -------------------------------------------------------- |
| Base            | —                                  | Every tier       | —                 | —                       | Tier 1 speed(base 1.0 move speed), no clash power        |
| Tap A           | Flick dash                         | (1–4 reach)      | 0.1s              | 1.0s (down per +1 tier) | No speed reset                                           |
| Tap B           | Parry                              | ≤ Tier 4         | 0.2s              | 1.0s (0.5s if < tier 1) | Stun 1s +0.5s/tier                                       |
| Hold A          | Accelerate                         | Tier 1→4         | Ramp slowly       | —                       | Turn rate ↓ per tier                                     |
| Hold B          | Decelerate                         | +1 momentum/2s   | Reduce rapidly    | —                       | Parry CD ↓                                               |
| Double Tap A    | Item use                           | —                | —                 | Item                    | No other button are being pressed                        |
| Double Tap B    | Skill use                          | —                | —                 | Skill                   | No other button are being pressed                        |
| Tap A + Tap B   | Dodge                              | Tier 1–4         | 0.2s              | 5s                      | Loses to higher tier                                     |
| Tap A + Hold B  | Momentum dash + back shocktrail    | Tier 1–4         | Dash 0.2s         | 2s                      | **Reset speed → Tier 1**                                 |
| Hold A + Tap B  | Hard stop + front shocktrail       | Tier 1–4         | Burst             | 2s                      | **Reset speed → Tier 1**                                 |
| Hold A + Hold B | Nuke charge dash + big shock trail | Tier 5 at 5 sec  | Charge infinite   | —                       | Unparry-able, **reset → Tier 0**, speed 0 while charging |

TO-DO:
Add zoom level to menu, tweak every thing to be diegetic.
Overhaul gameplay.(heat, scores)

Check core code structure again
Add contents (base)
Add sounds
Continue...