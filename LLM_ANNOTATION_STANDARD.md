# LLM Annotation Standard

This file makes `scene.json` output look the same no matter which LLM (Claude, ChatGPT, Gemini, etc.) writes it. The [README](README.md) explains *what* to annotate; this file fixes *how* the JSON is written: exact keys, value formats, vocabulary, and length limits.

To use it, paste the **Prompt** section below into the model along with the scene photos and your measurements. Then check the output against the **Checklist** before adding it to `Data/`.

---

## Prompt

Copy everything inside this block into the model:

~~~text
You are annotating one scene for the Manipulation Scene Bank. Output ONE JSON object and nothing else:
no Markdown fences, no comments, no text before or after it.

Inputs I will give you:
- Scene ID (e.g. S0031) and 1+ photos of the same unchanged setup, in order (01, 02, 03...).
- Object measurements in cm, if I have them.
- Optionally: "Staged permutation of <scene IDs>" if the same objects appear in other scenes.

Follow these rules exactly:

STRUCTURE
- Keys and key order must match the TEMPLATE below exactly. Do not add, rename, or drop keys.
- 2-space indentation, double quotes, no trailing commas, UTF-8.
- "images" lists every photo as "images/01.jpg", "images/02.jpg", ... (lowercase .jpg, two digits).
- target_id values are "T001", "T002", ... in order, with no gaps.

VALUES
- Unknown scalar -> null. Empty list -> []. Yes/no -> true/false (never strings).
- Dimensions are in METERS (12 cm -> 0.12). Record them as measured with the object upright,
  even if it is lying on its side in this scene.
- measurement_type: "measured" (I measured it), "known" (spec sheet/label), "estimated"
  (judged from the photo), or "unknown" (then width/depth/height are all null).
- distance_cm: a number only if you are at least 80% confident, using known object sizes as a
  ruler. Otherwise null. Never guess to fill the field.
- generic_improvement_needed is true if and only if candidate_improvements is non-empty.
- physically_blocked is true only if another object rests on or fully blocks the target.

VOCABULARY
- manipulation_factors may only use:
  far_from_accessible_edge, low_side_clearance, low_overhead_clearance, against_wall,
  partially_obstructed, fully_obstructed, under_another_object, surrounded_by_clutter,
  awkward_orientation, limited_approach_space
  If nothing applies, use [] (do not use open_clearance or isolated_target).
- action may only be: translate_target, rotate_target, move_obstacle, reconfigure_target.
- direction is snake_case and starts with one of:
  left, right, toward_front, toward_back, away_from_wall, away_from_<object>,
  off_<object>, out_of_<container>, toward_center
  e.g. "toward_front_of_shelf", "away_from_red_can". Use null for rotations or when no
  single direction fits.
- orientation starts with one of: "upright", "lying flat", "lying on its side",
  "upside down", "leaning", "tilted", "tipped up", optionally followed by ", <short detail>".
- source is exactly "locally collected scene", or
  "Staged permutation of S00XX and S00YY, using the same objects in a different arrangement."

WRITING
- Directions (front/back/left/right) are as seen facing the surface; front is its open,
  accessible edge. Say this once at the end of scene_description. Never refer to a
  specific photo ("in 01.jpg", "in the second image").
- Only mention objects that are targets, plus fixed structures that affect access (walls,
  panels, shelf rails). Leave out background clutter, items on other shelves, people, etc.
- Each target "description" must single out that object: color + type + label text if
  any + position relative to the surface and neighboring objects.
- Length limits (characters, including spaces):
    scene_description        <= 300  (2-4 sentences)
    target description       <= 200  (1 sentence)
    clearance_notes          <= 120  (1 sentence)
    improvement description  <= 120  (1 sentence, starts with a verb, includes the distance
                                      as "approximately N cm" when distance_cm is set)
    other_notes              <= 80   ("" if there is nothing important to add)
- Do not invent problems. Easy targets get manipulation_factors [],
  generic_improvement_needed false, candidate_improvements [].
- Do not reason about a specific robot's reach, payload, or gripper size.

TEMPLATE
{
  "scene_id": "S0000",
  "images": [
    "images/01.jpg"
  ],
  "scene_description": "",
  "targets": [
    {
      "target_id": "T001",
      "name": "",
      "description": "",
      "approximate_dimensions_m": {
        "width": null,
        "depth": null,
        "height": null,
        "measurement_type": "unknown"
      },
      "orientation": "",
      "nearby_obstacles": [],
      "physically_blocked": false,
      "clearance_notes": "",
      "manipulation_factors": [],
      "generic_improvement_needed": false,
      "candidate_improvements": [
        {
          "action": "translate_target",
          "direction": "toward_front_of_shelf",
          "distance_cm": null,
          "description": ""
        }
      ],
      "other_notes": ""
    }
  ],
  "source": "locally collected scene"
}
~~~

---

## Example target

From [Data/S0021/scene.json](Data/S0021/scene.json):

~~~json
{
  "target_id": "T002",
  "name": "purple coffee can",
  "description": "Unopened purple 'Chameleon Latte classic' can in the back-left corner, standing directly in front of the leaning notebook.",
  "approximate_dimensions_m": {
    "width": 0.05,
    "depth": 0.05,
    "height": 0.14,
    "measurement_type": "measured"
  },
  "orientation": "upright",
  "nearby_obstacles": [
    "Notebook touching it from behind",
    "White left panel about 2 cm to its left"
  ],
  "physically_blocked": false,
  "clearance_notes": "Open to the front and right, but boxed in by the notebook behind and the panel on its left.",
  "manipulation_factors": [
    "limited_approach_space",
    "low_side_clearance",
    "far_from_accessible_edge"
  ],
  "generic_improvement_needed": true,
  "candidate_improvements": [
    {
      "action": "translate_target",
      "direction": "toward_front_of_desk",
      "distance_cm": 10,
      "description": "Move the purple can approximately 10 cm toward the front of the desk, out of the corner."
    }
  ],
  "other_notes": ""
}
~~~

---

## Files and folders

~~~text
Data/
  S0031/
    scene.json
    images/
      01.jpg
      02.jpg
      03.jpg
~~~

- The folder name, `scene_id`, and the `images` list must all match.
- Photos are renamed to `01.jpg`, `02.jpg`, ... in the order they were taken. Use lowercase `.jpg`; convert iPhone `.HEIC` photos to JPG first.
- Store each photo once, even if it has several targets.

---

## Checklist

Before adding a scene, check that:

- [ ] The output is valid JSON with exactly the template's keys, in the template's order.
- [ ] `scene_id` matches the folder name, and `images` matches the files in `images/`.
- [ ] Target IDs run `T001`, `T002`, ... with no gaps.
- [ ] Dimensions are in meters, and `measurement_type` says where they came from.
- [ ] Every `manipulation_factors` entry and `action` comes from the allowed lists.
- [ ] `generic_improvement_needed` is `true` exactly when there are candidate improvements.
- [ ] Every number in `distance_cm` is one you would bet on; otherwise it is `null`.
- [ ] No text field goes over its length limit, and nothing refers to a specific photo.
- [ ] Background objects that aren't targets aren't described.
- [ ] Easy targets are left as easy, with no invented problems.
