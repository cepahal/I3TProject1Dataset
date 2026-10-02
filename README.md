# I3TProject1Dataset
scene lists with json files and pngs
# Robot Manipulation Dataset

## Goal

Collect many images of environments containing objects that could later be designated as **target objects** for a robot to grab or otherwise manipulate. The purpose of this dataset is to create a large bank of manipulation situations that can later be combined with many different robots with different designated capabilities.

Note that all of these images should be taken from the perspective of a person working with a robot, **not from the robot's perspective**. Imagine this as if your Roomba has gotten stuck, it alerts you that something is wrong, and you go over and look to see what is wrong. The pictures we want are snapshots from your perspective at moments like that.

You can take these anywhere at any time. For example, when you're at dinner and you finish eating and place your plate in the middle of the table, imagine if a robot had to come pick that up. It would be much easier for it if the plate were near the edge of the table. A picture like that, with the plate in the middle of the table, is exactly what we want.

For generation of the descriptions below, you CAN use frontier models to help you out. Their outputs are generally reliable enough to work here, and should make the process go faster. A useful prompt to give a model initially can be found at the end of this document.

---

## Important: One Image Can Produce Multiple Examples

A single scene image may contain several useful objects. For example, one shelf image might contain:

- a can;
- a book;
- a bottle;
- a box.

That same image can later be used as four separate training examples:

- Image S0012 → target = red can at the rear-left side of the middle shelf
- Image S0012 → target = blue book lying flat near the right wall of the shelf
- Image S0012 → target = clear bottle directly in front of the book
- Image S0012 → target = cardboard box at the front-left corner of the shelf

Each target should receive its own target annotation in the scene JSON.

Because we are **not requiring bounding boxes or other image annotations**, target descriptions need to be specific enough that someone looking at the image can tell exactly which object is being referenced.

For example:

**Bad:**

> "can"

**Good:**

> "Red aluminum can at the back-left side of the middle shelf, immediately to the left of the clear bottle."

If there are multiple similar objects in an image, use color, position, nearby objects, or other visible characteristics to clearly distinguish the target.

Reusing images in this way is encouraged because it allows us to get substantially more training examples from the same scene while keeping the visual environment constant.

---

## Scene Collection

Create as much variation as possible.

Useful settings include:

- shelves
- tables
- counters
- cabinets
- storage areas
- workbenches
- cluttered surfaces
- bins and containers

Useful objects include:

- cans
- books
- bottles
- boxes
- tools
- containers
- household objects
- pretty much anything you can think of or run into

---

## Vary Manipulation-Relevant Factors

Look for or create scenes involving different:

- object distances from the edge or accessible portion of the surface they are on;
- object heights relative to the floor;
- orientations;
- proximity to walls;
- amounts of clutter surrounding them;
- obstacles (e.g., a target box is behind another box);
- available gripper clearance (is the target in a tight space?);
- object stacking (e.g., the target book is underneath another book).

We ultimately want scenes where grasping ranges from clearly easy to clearly difficult.

---

## Scene JSON

Each scene receives one `scene.json`.

Example:

~~~json
{
  "scene_id": "S0012",

  "images": [
    "images/01.jpg"
  ],

  "scene_description": "Shelf containing several household objects.",

  "targets": [
    {
      "target_id": "T001",

      "name": "red can",

      "description": "Red aluminum can at the rear-left side of the middle shelf, immediately to the left of the clear bottle.",

      "approximate_dimensions_m": {
        "width": null,
        "depth": null,
        "height": null,
        "measurement_type": "unknown"
      },

      "orientation": "upright",

      "nearby_obstacles": [
        "Clear bottle immediately to the right of the can"
      ],

      "physically_blocked": false,

      "clearance_notes": "Limited free space on the left side and substantial distance from the front edge of the shelf.",

      "manipulation_factors": [
        "far_from_accessible_edge",
        "low_side_clearance"
      ],

      "generic_improvement_needed": true,

      "candidate_improvements": [
        {
          "action": "translate_target",
          "direction": "toward_front_of_shelf",
          "distance_cm": 10,
          "description": "Move the can approximately 10 cm toward the front of the shelf."
        },
        {
          "action": "translate_target",
          "direction": "right",
          "distance_cm": 5,
          "description": "Move the can approximately 5 cm to the right to increase surrounding clearance."
        }
      ],

      "other_notes": ""
    },

    {
      "target_id": "T002",

      "name": "blue book",

      "description": "Blue book lying flat near the rear-right side of the middle shelf, directly against the rear wall.",

      "approximate_dimensions_m": {
        "width": null,
        "depth": null,
        "height": null,
        "measurement_type": "unknown"
      },

      "orientation": "flat",

      "nearby_obstacles": [
        "Shelf wall directly behind the book"
      ],

      "physically_blocked": false,

      "clearance_notes": "Very little clearance behind or underneath the book.",

      "manipulation_factors": [
        "low_clearance",
        "awkward_orientation",
        "against_wall"
      ],

      "generic_improvement_needed": true,

      "candidate_improvements": [
        {
          "action": "translate_target",
          "direction": "toward_front_of_shelf",
          "distance_cm": 10,
          "description": "Move the book approximately 10 cm toward the front of the shelf."
        },
        {
          "action": "rotate_target",
          "direction": null,
          "distance_cm": null,
          "description": "Place the book upright with its spine facing outward."
        }
      ],

      "other_notes": ""
    },

    {
      "target_id": "T003",

      "name": "green bottle",

      "description": "Green bottle standing alone at the front-center of the bottom shelf with open space on all sides.",

      "approximate_dimensions_m": {
        "width": null,
        "depth": null,
        "height": null,
        "measurement_type": "unknown"
      },

      "orientation": "upright",

      "nearby_obstacles": [],

      "physically_blocked": false,

      "clearance_notes": "Substantial open space around the bottle.",

      "manipulation_factors": [],

      "generic_improvement_needed": false,

      "candidate_improvements": [],

      "other_notes": "No obvious environmental change is needed to improve general manipulation accessibility."
    }
  ],

  "source": "locally staged scene"
}
~~~

The important idea is:

> **One scene → one or more images → multiple possible target annotations.**

---

## What Should Be Recorded About a Target?

Keep the target annotation simple.

Record:

- a unique target ID;
- a clear name;
- a **specific description that uniquely identifies the target within the image**;
- a short description of where it is relative to the surface it is placed on and surrounding objects;
- approximate dimensions, if known;
- whether those dimensions were measured, already known, estimated, or are unknown;
- qualitative orientation (e.g., lying flat, upright, tilted);
- nearby obstacles;
- whether another object physically blocks it;
- useful notes about free space or clearance;
- manipulation-relevant factors visible in the scene;
- candidate changes that could make the object generally easier to manipulate;
- anything unusual that may affect manipulation.

### Measurement Type

For approximate dimensions, use one of:

- `"measured"` — physically measured by the collector;
- `"known"` — obtained from a reliable known specification;
- `"estimated"` — visually approximated;
- `"unknown"` — dimensions are not available.

Do not invent measurements simply to fill the field.

---

# Manipulation Factors

The `manipulation_factors` field records **what about the current physical setup may make manipulation easier or harder**.

This should describe the current scene, not a particular robot's capabilities.

Examples include:

~~~json
"manipulation_factors": [
  "against_wall",
  "low_side_clearance",
  "awkward_orientation"
]
~~~

Useful labels may include:

- `far_from_accessible_edge`
- `low_side_clearance`
- `low_overhead_clearance`
- `against_wall`
- `partially_obstructed`
- `fully_obstructed`
- `under_another_object`
- `surrounded_by_clutter`
- `awkward_orientation`
- `limited_approach_space`
- `open_clearance`
- `isolated_target`

Multiple factors may apply to the same target.

If there are no obvious manipulation issues, use:

~~~json
"manipulation_factors": []
~~~

---

# Manipulation Improvement Field

The `candidate_improvements` field provides possible answers to the question:

> **"How could the physical configuration of this target be changed to make it generally easier for a robot to manipulate?"**

This is **not a robot-specific ground-truth solution**. Different robots may have different needs. These are reasonable environmental changes that appear likely to improve general manipulation accessibility.

There may be **more than one valid improvement**, so this field should always be a list.

Each improvement should contain:

- the type of physical action;
- the direction, if relevant;
- an approximate distance, if it can reasonably be determined;
- a short natural-language description.

If the distance is not known, use `null`.

---

### Position

**Before:**  
Can near rear of shelf.

**Possible solution:**  
Move can 15 cm toward the front of the shelf.

~~~json
{
  "action": "translate_target",
  "direction": "toward_front_of_shelf",
  "distance_cm": 15,
  "description": "Move the can approximately 15 cm toward the front of the shelf."
}
~~~

---

### Orientation

**Before:**  
Book lying flat.

**Possible solution:**  
Rotate book upright with spine outward.

~~~json
{
  "action": "rotate_target",
  "direction": null,
  "distance_cm": null,
  "description": "Place the book upright with its spine facing outward."
}
~~~

---

### Clearance

**Before:**  
Bottle directly against a wall.

**Possible solution:**  
Move bottle at least 5 cm away from the wall.

~~~json
{
  "action": "translate_target",
  "direction": "away_from_wall",
  "distance_cm": 5,
  "description": "Move the bottle at least 5 cm away from the wall to create additional clearance."
}
~~~

---

### Obstruction

**Before:**  
Target blocked by another object.

**Possible solution:**  
Move the blocking object so the target is accessible.

~~~json
{
  "action": "move_obstacle",
  "direction": "right",
  "distance_cm": null,
  "description": "Move the blocking object to the right until the target is no longer obstructed."
}
~~~

---

## Scenes That Do Not Need Improvement

Do **not** invent a problem for every target.

If an object already appears to be in a good configuration, record:

~~~json
"manipulation_factors": [],
"generic_improvement_needed": false,
"candidate_improvements": []
~~~

For example, an isolated bottle near the front of a shelf with substantial open space around it may not need any obvious environmental change.

This is important because the final dataset should contain examples where the correct answer is effectively:

> **No environmental change appears necessary.**

---

# Do Not Only Collect Obviously Bad Scenes

The scene bank should contain a wide range of setups. Include scenes that appear:

### Clearly Easy

- isolated object;
- large amount of free space;
- convenient orientation;
- little surrounding clutter.

### Borderline

- object near a wall;
- limited clearance;
- unusual orientation;
- partially obstructed target.

### Clearly Difficult

- heavily blocked object;
- object underneath another object;
- extremely constrained access;
- significant surrounding clutter.

The goal is simply to obtain diverse manipulation situations.

# Data Storage

Store each scene in its own folder using a unique Scene ID.

~~~text
dataset_b/

    S0001/
        scene.json
        images/
            01.jpg
            02.jpg

    S0002/
        scene.json
        images/
            01.jpg
~~~

Each scene folder should contain:

- one `scene.json` file;
- an `images/` folder containing all images for that scene.

## Scene IDs

Use sequential Scene IDs:

~~~text
S0001
S0002
S0003
...
~~~

The folder name and `"scene_id"` inside `scene.json` should match.

## Image Naming

Name images sequentially:

~~~text
01.jpg
02.jpg
03.jpg
...
~~~

List those same files inside `scene.json`:

~~~json
"images": [
  "images/01.jpg",
  "images/02.jpg"
]
~~~

## Multiple Targets

Do **not** make separate copies of an image for different target objects.

If one image contains several useful targets, store the image once and include multiple entries in the `"targets"` list.

Target IDs only need to be unique within that scene:

~~~text
T001
T002
T003
...
~~~

For example, the unique identifier for one target would effectively be:

~~~text
S0012 / T002
~~~

## Unknown Information

If information is not known, use `null` rather than guessing.

Example:

~~~json
"height": null
~~~

For list fields with nothing to record, use an empty list:

~~~json
"nearby_obstacles": []
~~~

For yes/no fields, use JSON booleans:

~~~json
"physically_blocked": false
~~~

The main goal is simply to keep every scene organized in the same way so the dataset can be processed automatically later.

---

# Frontier Model Prompt

# Dataset Annotation Summary for Image Analysis

You are helping create structured annotations for the **Manipulation Scene Bank**.

The dataset specification sheet defines the required JSON schema and field names. Follow that schema exactly. Your job is to look at the provided scene image(s), identify useful manipulation targets, and fill in the structured scene and target descriptions.

## Overall Purpose

The dataset contains images of everyday environments from the **perspective of a human collaborating with a robot**, not from the robot's camera.

Think of the image as representing what a person might see after walking over to inspect a situation where a robot may need help manipulating an object.

The goal is to describe:

1. what objects in the scene could reasonably serve as manipulation targets;
2. where each target is located relative to its supporting surface and surrounding objects;
3. what visible environmental factors could make manipulation easier or harder;
4. what physical changes could plausibly make the target easier for a generic robot manipulator to access or grasp.

The dataset is **not yet robot-specific**. Do not assume a particular robot, reach, gripper, payload, or kinematic capability unless that information is explicitly provided.

---

# Identifying Targets

A single image can contain **multiple useful target objects**.

For example, one shelf image may contain:

- a can;
- a book;
- a bottle;
- a box.

Each can become its own target entry in the same `scene.json`.

Prefer objects for which the surrounding scene creates a meaningful manipulation situation, whether easy, borderline, or difficult.

Do not create duplicate copies of the image for different targets.

---

# Target Descriptions Must Be Very Specific

Bounding boxes or segmentation annotations are not required.

Because of this, the textual target description must uniquely identify the intended object in the image.

Do not write:

> "can"

Instead write something like:

> "Red aluminum can at the rear-left side of the middle shelf, immediately to the left of the clear bottle."

Use visible properties such as:

- color;
- object type;
- position on the surface;
- nearby objects;
- relative position;
- orientation.

If multiple similar objects are present, the description must make it clear exactly which one is the target.

---

# Describe What Is Visible, Not What You Assume

Record observable properties of the scene.

Examples include:

- object orientation;
- proximity to walls or edges;
- clutter;
- nearby obstacles;
- whether another object physically blocks the target;
- clearance around the target;
- whether the target is underneath or behind something;
- whether there is substantial open space around it.

Do not invent exact measurements from the image.

If a measurement is not known, use `null`.

If approximate dimensions are visually estimated, mark the measurement type as `"estimated"` rather than presenting the value as measured fact.

---

# Manipulation Factors

For each target, identify visible properties of the current setup that may affect generic robot manipulation.

Examples include:

- `far_from_accessible_edge`
- `low_side_clearance`
- `low_overhead_clearance`
- `against_wall`
- `partially_obstructed`
- `fully_obstructed`
- `under_another_object`
- `surrounded_by_clutter`
- `awkward_orientation`
- `limited_approach_space`
- `open_clearance`
- `isolated_target`

Use multiple factors when appropriate.

These factors describe the **environmental configuration**, not the limitations of a particular robot.

Do not say something is outside a robot's reach, too heavy, too wide for a gripper, etc. unless a specific robot and its capabilities are explicitly provided.

---

# Candidate Improvements

For targets where the physical setup appears improvable, provide one or more **candidate improvements**.

These should answer:

> "How could the physical configuration of this target or its surroundings be changed to make it generally easier for a robot to manipulate?"

Examples:

- move the target closer to the accessible edge of a shelf;
- rotate a book upright;
- move an object away from a wall;
- move a blocking object aside;
- create more clearance around the target;
- expose a larger graspable surface.

Each candidate improvement should include:

- the action;
- direction when relevant;
- approximate distance only when reasonably known;
- a concise natural-language description.

There may be several valid improvements. Record multiple candidates when appropriate.

These are **generic candidate improvements**, not robot-specific ground-truth solutions.

---

# Do Not Invent Problems

The dataset must contain easy scenes as well as difficult ones.

If a target already appears easy to access and manipulate:

- leave `manipulation_factors` empty if there are no meaningful issues;
- set `generic_improvement_needed` to `false`;
- set `candidate_improvements` to an empty list.

Do not create an unnecessary correction merely because the schema contains an improvement field.

A valid annotation may conclude that no environmental change is obviously needed.

---

# Difficulty Diversity

Useful targets may appear:

## Clearly Easy
Examples:
- isolated object;
- open space on all sides;
- convenient orientation;
- little clutter.

## Borderline
Examples:
- close to a wall;
- limited clearance;
- unusual orientation;
- partly obstructed;
- farther from an accessible edge.

## Clearly Difficult
Examples:
- heavily obstructed;
- underneath another object;
- tightly constrained;
- surrounded by clutter.

Do not formally assign these categories unless the schema asks for them. They simply describe the range of examples we want in the dataset.

---

# Multiple Images

If several images show the same physical scene, they may all belong to the same scene entry.

Use all provided views to create the best possible description of the environment and targets.

Do not treat different camera views of the same unchanged setup as separate scenes unless instructed to do so.

---

# JSON Rules

Follow the dataset specification sheet exactly.

Important conventions:

- unknown scalar values → `null`
- empty lists → `[]`
- yes/no values → JSON booleans such as `true` and `false`
- do not replace unknown values with guesses
- use the required Scene ID and Target ID formats
- keep all target entries within the same scene JSON when they come from the same scene

The output should be valid JSON and should require as little manual cleanup as possible.

---

# Core Principle

For every image, reason in this order:

1. **What objects could be useful manipulation targets?**
2. **How can each target be described so precisely that someone can identify it without a bounding box?**
3. **What is visibly true about its position, orientation, clearance, and surrounding environment?**
4. **What aspects of that setup could help or hinder generic robot manipulation?**
5. **Would any physical environmental change plausibly make manipulation easier?**
6. **If so, what change should be made?**
7. **If not, explicitly record that no obvious improvement is needed.**

Do not perform robot-specific feasibility reasoning unless a robot capability description is explicitly supplied.
