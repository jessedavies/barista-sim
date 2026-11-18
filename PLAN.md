# Barista game — plan

Working title: **Shaky Hands** (alternatives: *Short Black*, *Rush Hour*, *Extraction*).
A first-person barista game. You control the barista's hands directly and clumsily, Surgeon Simulator style. The challenge is speed, Overcooked style: dockets pile up during the 7:30am rush at a high-end Melbourne cafe.

Status: planning, 2026-11-17. No code yet.

---

## The one feeling

**Grace under pressure, earned.** At first you're clumsy and the rush wins. Over a few shifts your hands get smooth, and you're pulling a flat white with a heart while three dockets wait. The comedy comes from clumsiness. The reward comes from getting good.

### The core tension, resolved up front

Surgeon Simulator is fun because you fight your own hands. Overcooked is fun because time runs out. Put the two together and the danger is frustration: you're too clumsy to ever be fast.

**Rule: clumsy but learnable, and every mistake is cheap and funny.**
- The controls are awkward but *consistent*. The same input always gives the same result, so practice builds real skill. There's no random wobble.
- A spill or a dropped cup costs you **seconds, not the order**. Grab a fresh cup and go again. A mistake should make you laugh, not restart.
- One "clumsiness dial" (spring stiffness, damping, grip tolerance) sets the difficulty. Tune it in the toys before anything else exists.

---

## Engine: Three.js + Rapier, in the browser

**Recommendation:** Three.js, Rapier physics (`@dimforge/rapier3d-compat`, WASM), TypeScript, and esbuild. Copy the dev/build scaffold straight from Henstead (`build.mjs`, `dev.mjs`).

Why:
- **The agent loop works best in a browser.** Claude can run it, screenshot it, drive input and read the console in the built-in browser pane. dream-loop's visual critic and every game-dev skill pack below assume this setup. Godot and Unity editors are mostly hidden from an agent, and their web exports are heavy and slow to iterate on.
- **Rapier handles the hard part.** You get physics hands from joints and motors (hand driven by a spring toward the cursor, fingers as motorised joints), and objects like cups and jugs can switch between dynamic and kinematic when grabbed. Nobody has published a ready-made mouse-controlled version, so this is the main thing to build.
- **Lightest viable test.** Each toy is one HTML file. Henstead shows the stack can reach shipped quality.
- **We already learned the lighting lesson.** Henstead's "models don't feel polished" problem was really a lighting problem (DD-005). Use ACES tone mapping, soft shadows and a warm key light from day one.

Skip React Three Fiber, since a single-screen game gains nothing from React. **Reconsider the engine if** articulated hands need more than Rapier joints can give, or if this ever heads for Steam or console. Godot 4 would be the move then.

---

## Skills first

The goal is to have the feel and the look captured in reusable skills *before* building the game, so every session builds toward the same target.

### Adopt (install, then read before trusting)

| Skill | Use it for | Watch out for |
|---|---|---|
| [achimala/dream-loop](https://github.com/achimala/dream-loop) | Locking the **look**: generate a target screenshot of the barista's view, build until a vision critic agrees the game matches | Only tested in Codex. Needs an image model: use the Figma MCP's `generate_image`, which is already connected. Optimises visuals only, not gameplay. Always give it a time budget |
| [majidmanzarpour/threejs-game-skills](https://github.com/majidmanzarpour/threejs-game-skills) | Game architecture: a director skill that routes to gameplay systems, game UI and 3D generation. Comes with a Vite + TS scaffold | Use its patterns, not its scaffold. Keep Henstead's esbuild setup |
| [cloudai-x/threejs-skills](https://github.com/cloudai-x/threejs-skills) | Current Three.js API knowledge (materials, post-processing, DOF) | Overlaps with the pack above. Keep whichever is sharper |
| [gamedev-skills/awesome-gamedev-agent-skills](https://claudeskills.info/plugins/gamedev-skills/awesome-gamedev-agent-skills/) | Browse for game-feel and juice skills, and the Rapier/physics ones | 60+ skills of mixed quality. Cherry-pick, don't install them all |

Also worth a look: [opusgamelabs/game-creator](https://skillselion.com/skills/opusgamelabs/game-creator/make-game) is opinionated about EventBus and GameState structure, and the [r3f-physics skill](https://skillselion.com/skills/enzed/r3f-skills/r3f-physics) covers Rapier grab patterns.

### Write our own (this is where the feel lives)

| Skill | Scope | Contents |
|---|---|---|
| `cafe-look` | project | Art direction for a Melbourne specialty cafe from behind the bar. Materials: terrazzo, blonde timber, brushed steel, matte ceramic, brass. Henstead's lighting recipe. A foreground that's fully 3D (machine, grinder, bench, milk fridge) and a background that's a depth-of-field blur with customer silhouettes. Seeds the dream-loop target prompt. No real brands or real cafes |
| `hand-feel` | project | The feel spec. Every tunable (spring k, damping, finger curl speed, grip threshold) with its current value and the reason for it. Feel targets in words and numbers ("cup to group head in under 5s after 1 minute of practice"). A lil-gui tuning panel. A juice checklist: steam, the hiss and the paper-tearing sound of milk stretching, the clink of ceramic, a little screen shake on a drop |
| `order-book` | project | Australian orders as data: recipe steps, cup type, shot, milk and texture, plus a way to add an order. Includes docket names and modifiers (see below) |
| `feel-check` | **global** (`~/.claude/skills`) | The verify loop for any game: start the dev server, replay scripted input, capture a gif or screenshots, measure frame time, then run a short owner's-read questionnaire for your playtests. Reusable for Henstead from day one |

---

## Orders (the menu, as `order-book` data)

Core, roughly in unlock order:
1. **Long black**: hot water first, then a double shot on top to keep the crema
2. **Flat white**: double shot, thin silky microfoam, ceramic cup
3. **Latte**: in a glass, a bit more foam
4. **Cappuccino**: thick foam, chocolate dusted on top
5. **Piccolo**: ristretto topped with latte milk in a small glass
6. **Magic**: the Melbourne one. Double ristretto, flat-white milk, 5–6oz cup
7. **Macchiato**: short or long, with a dash of foam
8. **Mocha**, **chai latte**, **dirty chai**
9. **Babycino** with marshmallows (a joke order, and a breather)
10. Iced long black, affogato, espresso tonic (summer menu)

Modifiers stack difficulty without needing new mechanics: oat, soy, almond, lactose-free, extra hot, half strength, decaf, extra shot, keep cup, "weak", "large but in a small cup".
Dockets carry names. Regulars come back with the same order ("Dave, oat magic, extra hot") and remembering them earns a bonus.

---

## Build order

**1. Skills (week 1).** Install and vet the adopt list. Write `cafe-look` and `order-book`. Run dream-loop once to get the target screenshot of the barista's view. That image becomes the look we build toward.

**2. Toys (Tier 0).** One HTML file per mechanic. Each answers one question, and the hand toy comes first:
- **Hand**: mouse moves the hand, keys curl the fingers. Can you lift a cup and place it under the group head? Is a miss funny? *This toy decides whether the game exists.* Write `hand-feel` from what you tune here.
- **Portafilter**: dose, tamp, lock in with a twist. Does it feel physical?
- **Milk**: steam wand depth controls stretching versus texturing. Do the sound and the visible foam make it satisfying?
- **Pour**: can you pour a rough heart? Stretch goal. Leave it out of the slice if it isn't fun.

**3. Vertical slice.** One station, three orders (long black, flat white, magic), and a 3-minute rush with dockets on the rail. Scoring is speed plus quality (crema, milk texture, temperature). Design the peak and the end explicitly:
- **Peak**: handing a perfect flat white across the pass while dockets stack up behind you, and the customer reacts.
- **End**: the end-of-shift docket spike, e.g. "47 coffees, 2 jugs dropped, Dave's oat magic was perfect."

**4. Expand**, only if the slice is fun: more orders and modifiers, regulars, a rush that escalates across a week of shifts, and maybe a co-op mode later (two baristas, one machine — the Overcooked bit).

---

## Open questions

- **Control scheme.** Surgeon Simulator uses one key per finger, which is great for comedy but maybe too much at speed. Try both: a 4 fingers + thumb version, and a simpler grip/pinch version. The hand toy decides.
- **One hand or two?** Two hands is truer to a barista (jug in one, cup in the other) and doubles the comedy, but it also doubles the control load. Start with one hand and a "set down" bench.
- **Touch and iPad.** Desktop first. Henstead's iPad constraint doesn't carry over unless you want it to.
- **Audio** carries a lot of the feel here: grinder, knock box, steam and milk stretching. Sourcing CC0 audio needs planning.

---

## Personal-playbook checklist

1. **The one feeling, and does everything serve it?** Grace under pressure, earned. Every mechanic has to be awkward at first and learnable. Anything that stays random fails this test.
2. **The lightest tier that tests the next assumption?** A single HTML hand toy. If grabbing a cup isn't fun, nothing else matters.
3. **Where is the peak, and is the ending designed?** Both are designed on purpose (see the slice). The end-of-shift summary is the [[peak-end-rule]] ending, and its detail ("Dave's magic was perfect") makes the shift [[storyteller-bias]]-memorable.
4. **Which engagement mechanics are earned?** Unlocking the menu ([[collection-bias]]), regulars ([[surprise-effect]] in the content, never in the structure), and the [[humor-effect]] with a high budget. Spills and docket names are the jokes. Avoid timers that punish beyond the rush itself, and avoid fail states that wipe progress.
5. **Is the constraint protected from convenience creep?** The constraint is the clumsy hands. The pressure will be to add auto-snap and assists until it's just Overcooked. Allow assists only as a visible difficulty setting, never quietly.
6. **Who tried it and left?** Someone who couldn't pick up a cup in their first 30 seconds. The hand toy needs a cold playtest with someone who isn't you (one of the kids?) before the slice starts.

---

## Revision, 2026-11-17: the full coffee loop, with easy hands

Two changes came out of the first playtests.

**1. Controls: easy hands is the default, and Surgeon mode is a toggle (G).**
Five finger keys plus click-to-lower suit a single-cup comedy. They don't suit a full workflow, and pouring and steaming need smooth height and tilt. The new scheme:

- **Hold click:** the hand reaches down to whatever is under the cursor and grips it.
- **Release:** it sets the thing down and lets go.
- **Scroll:** sets hand height (steam wand depth, pour height).
- **Right-drag:** tilts the wrist to pour, and it eases back level when you let go.

The clumsiness now comes only from physics: spring lag, slosh, knocks and slips.

**2. Pressure comes from volume and resources, not raw speed** (proposed earlier, now folded in). Orders arrive in growing waves. Milk is finite. Heat and patience set the quality windows. Drinks cool on the pass.

### The loop, station by station (smallest version of each)

| Station | What you do with your hand | What it scores | Reuses |
|---|---|---|---|
| **Grinder** | Unlock the portafilter and hold its basket under the chute. Grounds pile up while you hold it there. | Dose: under-dosed is weak, overdosed spills grounds | Portafilter grab |
| **Tamp** | Set the portafilter on the tamp mat, grab the tamper, press it down into the basket | Level and firmness (press time) | Grab + lower |
| **Group** | Lock it in, put a cup under it, and the shot runs | Shot quality = dose × tamp | Lock-in, shot (built) |
| **Knock box** | Bang the used puck out before the next dose | Can't re-dose a dirty basket | Portafilter + collision |
| **Fridge → jug** | Grab the 2L milk bottle and tilt to pour into the jug | Pour the right amount. Milk runs out, and a trip to the fridge costs seconds | Tilt + pour |
| **Steam wand** | Hold the jug with the wand tip in the milk. Scroll sets depth: shallow stretches (paper-tearing hiss, foam rises), deep rolls (heats) | Texture and temperature (55–65°C). The pitch drops as it heats, and your hand shakes when it's too hot to hold. Over 70°C, that milk is wasted | Height + sound |
| **Pour** | Tilt the jug over the cup. Pouring high mixes, low lays foam on top | Foam depth decides the drink: flat white thin, latte medium, cap thick | Tilt + pour |
| **Takeaway** | The docket says TA: take a paper cup from the stack, and lid it at the end | Lid on, or it spills on the pass | Cup + place |

### Build order
1. **Espresso chain:** grinder, tamp, knock box. This reuses everything already built.
2. **Milk chain:** fridge, jug, wand, pour. This is the heart of the heat-and-patience loop.
3. **Takeaway cups and lids.**
4. **The load layer:** a docket rail, waves of orders, drinks cooling on the pass, and the end-of-shift summary.
