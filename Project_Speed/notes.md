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

**Comfortable set**: 
Tier 0 = stop, 1 = base speed, can't clash, only recieves damage, same speed tier clash, higher speed tier damages lower ones.
Tier 0= completely still, tier 0.5 = 0.5 base speed, Tier 1 (base)=1.0 speed, tier 2= 3.0, tier 3=6.0, tier 4=100.0 (for tier 4 unbreakable dash only).
Require smooth transition logic when transition from a button to a combo and vice versa
Clash requires both ≥ Tier 2
Tier 1 never clashes, only takes damage
* Tap A = short dash, 1s cd, tier 2
* Tap B = parry, stun parried enemy for 1 sec base, tier 2 parried=1s stun, tier 3=2s, stun (parry window 0,2s, cd 1s)
* Hold A = sprint mode (tier 2)
* Hold B =  x0,5 base speed, walk mode (tier 0.5) reduce 50% damage.
* Tap A + Tap B = (reserved for elite/boss skill copy later)
* Tap A + Hold B = dodge (iframe, completely untouchable for the same window and cd as parry, cannot dodge tier 4)
* Hold A + Tap B = reduce intertia immediately for sharp turn (1s cooldown) does not affect speed
* Hold A + Hold B = charge infinitely, speed drop to 0 during charge, (tier 3-4, tier 3-2s, tier 4=3s,tier 4 is unparryable but clashable) cannot be staggered if charging.

Speed tier will now changes dynamically based on current speed (From tier 0 to 5) 0 stop, 1 = base speed, can't clash, only recieves damage, same speed tier clash, higher speed tier damages lower ones. Make the physics more "realistic" now.
Tier 0= completely still, tier 0.5 = slowed down, Tier 1 (base)=1.0 speed, tier 2= 2.0, tier 3=3.0, tier 4=4.0, tier 5=5.0.
Clash requires both ≥ Tier 2
Tier 1 never clashes, only takes damage
Damage = relative speed delta, not flat tier

TO-DO:
Add zoom level to menu, tweak every thing to be diegetic, tweak visual FX
Overhaul gameplay.(heat, scores)
Better AI and enemy variants
Check core code structure again
Add contents (base)
Add sounds
Continue...

Finally, condense/minify up the codebase WITHOUT REMOVING ANYTHING. you can clean up the comments, put functions on the same line, etc..but do not remove any code. or at least you can check unused/legacy codes, but i doubt there's any, if you remove anything, report back to me.
 need you to implement these normal enemies, auto put in corresponding traits and tags  and expand the memory system for them.

Warrior: He is our typical "player" he knows when to commit and when to back out, but generally he will fight til his last breath to protect his teammates. He often hold A to sprint, release when charged enough to dash, and repeat. he can dodges, he can parry, although not often, he's a competent AI.
Tanker:He braces a lot, close in with the player, also parry from time to time although he's not too precise with it. When he recognizes his team have an archer, he will try to protect the archer by standing between the player and the archer.
Archer:stays far away, and will do charges a lot, sometimes charges to tier 4 too. after done charge and dash, he will return back to a safe position.
Thief:This mf is a troll, he circles around the player, feint in and out, to "steal" player's precious time and focus, he won't directly commit to a kill unless he's low HP and is the only guy left on the field. He also will focus on getting the loots more than others.
Berserker:pure respect, he won't back down no matter what, he mainly charges even from dangrous close distances, when he's low (<50% hp) he changes to constanty try to sprint nonstop at the player, never back out til he dies.

Also, standardize the naming, don't use names that no one will understand, be more "systemic" and uniform....with prefix, suffix, don't abreviate, etc...basically try to standardize the naming.
Also please fix the code to have no "magic numbers", i want to have everything configurable to be in the config zone.
ALso please get rid of the small and verbose comments  

