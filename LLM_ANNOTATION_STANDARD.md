# LLM Annotation Standard (SamARitan)

This file keeps `scene.json` output consistent no matter which LLM (Claude, ChatGPT, Gemini, etc.) writes it. It follows the *SamARitan Dataset Collection Instructions v3*: each scene holds training pairs that ask what a person should change, if anything, so that an upcoming or ongoing human–robot interaction goes safely and reliably. Scenes S0001–S0005 are the reference examples.

To use it, paste the **Prompt** below into the model together with the scene photos and any facts you know (measurements, staged distances, collector notes). Before adding the result to `Data/`, check it against the **Checklist**.

---

## Prompt

Copy everything inside this block into the model:

~~~text
You are writing one scene.json for the SamARitan dataset. Output ONE JSON object and nothing else:
no Markdown fences, no comments, no text before or after it.

GOAL
Each example is a training pair. The input is what a vision-language model sees at run time (one
photo plus task, robot and status context). The output is an ordered list of guidance specs: the
small physical changes a person should make now to prevent a foreseeable problem, or to fix one
that is already happening. If no change is needed, the list is empty ([]). Look for problems the
robot might not notice or could only fix at much greater cost: spills, cords pulling equipment,
objects being knocked over, blocked grasps, hidden labels, unstable stacks.

INPUTS I WILL GIVE YOU
- Scene ID (e.g. S0046) and 1+ photos of the same unchanged scene, named images/01.jpg, 02.jpg...
- Optional: measured object sizes, staged or measured move distances, and collector notes.

STRUCTURE (keys in exactly this order)
{
  "scene_id": "S0046",
  "images": ["images/01.jpg", "images/02.jpg"],
  "scene_description": "<1-3 plain sentences naming the visible objects and where they are>",
  "examples": [
    {
      "example_id": "E001",
      "vlm_input": {
        "image": "images/01.jpg",
        "question": "<QUESTION, copied exactly>",
        "task_context": "<what the robot will do next>",
        "robot_context": "<ROBOT BASE SENTENCE> <task-specific capability assumptions>",
        "status_context": null
      },
      "vlm_output": { "guidance_specs": [] },
      "review": {
        "phase": "before",
        "category": "efficacy_remediation",
        "problem": "<the foreseeable issue, or that none requires a change>",
        "rationale": "<why this change, or no change, is right>",
        "information_sources": ["headset_visual", "task_context", "robot_context"],
        "notes": "<NOTES BASE> <legacy/provenance details>"
      }
    }
  ]
}

QUESTION (use this exact text in every example):
Given the image and the task, robot, and status context, what should the human change, if anything,
to prevent or resolve problems with this interaction? Return only a JSON object containing an ordered
guidance_specs array; use an empty array if no intervention is needed.

ROBOT BASE SENTENCE (first sentence of every robot_context):
The robot has one claw gripper with parallel jaws and manipulates one object at a time.

NOTES BASE (first sentences of every review.notes):
Task and robot contexts are authored scenario assumptions, not recorded robot observations. All
listed photographs were visually reviewed; this example uses only the named input image. Robot
hardware follows the dataset-wide assumption of one claw gripper with parallel jaws.

WHAT GOES IN THE INPUT
- task_context: a specific task. Say what will happen to which object, where it goes, and what stays
  put. Name the robot's target, but not the object the person should change.
- robot_context: the base sentence, then capability facts the task depends on (how it grips, which
  side it approaches from, what it cannot do, such as unplug cords or operate caps while holding).
- status_context: null before the task. For a "during" example, give the assumed task facts
  (e.g. "The robot paused after its jaw touched the tape roll") and add "status_context" to
  information_sources.
- NEVER put the problem, the fix, or the reason for intervening in the input.

GUIDANCE SPECS (the output)
Each spec is {"target_object", "action": {"type", "qualifiers"}, "constraints", "context"}.
- target_object: the object the PERSON acts on (often not the robot's target). It must uniquely
  identify a visible object: color, type, label text, position, neighbors.
- action.type must be one of these six. Do not use any other type.
    move     {"direction": "left|right|forward|backward|up|down", "distance": <meters>, "unit": "meters"}
    rotate   {"axis": "x|y|z", "angle": <degrees>, "unit": "degrees",
              "direction": "clockwise|counterclockwise", "viewing_convention": "from_positive_axis"}
    interact {"operation": "<e.g. switch_on, switch_off>"}       e.g. turn on a light switch
    close    {}                                                   e.g. close a box lid or a flip cap
    clean    {"method": "<e.g. wipe>", "end_state": "<e.g. dry>"} e.g. wipe up a spill on the floor
    detach   {"reference_object": "<what it is detached from>"}   e.g. unplug a cord from an outlet
- Directions are global, along the captured camera heading of the named input image (photo 01 is
  always the front view), gravity-aligned, matching S0001-S0005: right/left = camera right/left;
  backward = toward the camera (the person's side); forward = away from the camera, along its
  heading; up/down = gravity. This holds even when the surface sits at an angle in the photo:
  "toward the front edge of the table" is "backward". For rotations, x = camera right, y = up,
  z = along the camera heading (away from the camera).
- One spec carries one direction. Describe a diagonal move as two separate specs (e.g. left, then
  backward), and lifting something off a stack as a sideways move plus a down move.
- constraints: short, descriptive (e.g. "Keep the bottle upright"). They are not executed.
- context: one sentence on why this step helps the upcoming interaction.
- List several specs in the order the person should do them (e.g. close the cap, then move the
  bottle; unplug the cord, then move the sharpener).

NUMBERS
- Give a distance or angle only when it is defensible: measured, deliberately staged, or derived
  from measured sizes. Never invent meters or degrees from the photo alone.
- If a move is needed but no defensible distance exists, set "distance": null and say so in notes.
  The collector must supply a value before the example is used with the composer.

REVIEW LABELS (dataset-only; never shown to the model)
- phase: "before" or "during".
- category: "efficacy_remediation" (task success, blocked grasp, retry), "safety_from_robot"
  (the robot could spill, knock over or drag something), or "safety_for_robot".
- problem / rationale: one or two sentences each.
- notes: the NOTES BASE, then where the target and distances came from (e.g. "Legacy T003, 10 cm
  toward the camera, encoded backward"), how directions were encoded, and any collector notes.

EXAMPLES TO INCLUDE PER SCENE
- At least one example per useful target, each with a specific task.
- No-change examples are required: the same object can need a fix for one task and none for
  another (e.g. carrying an open box vs. picking one item from it; non-contact reading tasks).
- Where it fits, add a "during" example that uses status_context.
- Don't invent problems because a scene is cluttered, and don't describe background objects that
  play no part in any task.
~~~

---

## Example

From [Data/S0025/scene.json](Data/S0025/scene.json): a plugged-in pencil sharpener stands in the path of a tray the robot will pull forward.

~~~json
{
  "example_id": "E004",
  "vlm_input": {
    "image": "images/01.jpg",
    "question": "Given the image and the task, robot, and status context, what should the human change, if anything, to prevent or resolve problems with this interaction? Return only a JSON object containing an ordered guidance_specs array; use an empty array if no intervention is needed.",
    "task_context": "The robot will pull the letter tray of paper straight forward off the wall and carry it level to a printer station.",
    "robot_context": "The robot has one claw gripper with parallel jaws and manipulates one object at a time. It grips the tray's front edge and drags it straight toward the front before lifting; it cannot steer around objects in that path.",
    "status_context": null
  },
  "vlm_output": {
    "guidance_specs": [
      {
        "target_object": "power plug on the end of the electric pencil sharpener's black cord",
        "action": {"type": "detach", "qualifiers": {"reference_object": "wall outlet the sharpener's cord runs to beyond the right end of the table"}},
        "constraints": ["Pull the plug itself, not the cord", "Leave the sharpener standing where it is"],
        "context": "Unplug the sharpener before it is moved so its cord cannot drag across the table."
      },
      {
        "target_object": "black electric pencil sharpener with a clear shavings bin in front of the letter tray",
        "action": {"type": "move", "qualifiers": {"direction": "right", "distance": 0.15, "unit": "meters"}},
        "constraints": ["Keep it upright so the shavings stay in the bin"],
        "context": "Move the sharpener out of the tray's forward path."
      }
    ]
  },
  "review": {
    "phase": "before",
    "category": "safety_from_robot",
    "problem": "The pencil sharpener stands in front of the tray's right half, in the robot's straight pull path.",
    "rationale": "Moving the small sharpener aside is easier than re-planning the pull; it must be unplugged first.",
    "information_sources": ["headset_visual", "task_context", "robot_context"],
    "notes": "Task and robot contexts are authored scenario assumptions, not recorded robot observations. ... Legacy T004, 15 cm right. ..."
  }
}
~~~

---

## Files and folders

~~~text
Data/
  S0046/
    scene.json
    images/
      01.jpg
      02.jpg
~~~

- The folder name and `scene_id` match, and `images` lists every file in `images/`.
- Name photos `01.jpg`, `02.jpg`, ... in the order they were taken, in lowercase `.jpg`. Convert iPhone `.HEIC` photos first.
- Store only photos of the initial, unchanged scene. Don't store after photos.

---

## Checklist

- [ ] Valid JSON with exactly the keys above, in that order.
- [ ] Every example's `question` matches the fixed text, and its `image` is one of the scene's photos.
- [ ] Every `robot_context` starts with the robot base sentence, and every `notes` starts with the notes base.
- [ ] No input field states the problem, the fix, or the reason to intervene.
- [ ] Every `action.type` is `move`, `rotate`, `interact`, `close`, `clean` or `detach`, with the qualifiers shown.
- [ ] Directions are from the person's viewpoint in the named image (toward the camera = `backward`, as in S0001–S0005), and diagonal moves are split into two specs.
- [ ] Every distance and angle is measured, staged or derived from measurements; any `null` is explained in the notes.
- [ ] Multi-step fixes are in the order the person should do them.
- [ ] The scene includes no-change examples, and `status_context` is filled in exactly when `phase` is `during`.
- [ ] Every target and reference object is uniquely identifiable in the named image.
