# d-questionnaire

interactive multi step questionnaire composing progress tracking, radio buttons, checkboxes, text inputs, and keyboard shortcuts.

```html
<d-questionnaire name="survey">
  <div d-step name="role" title="what is your role?" type="single" required>
    <div d-option value="dev" text="developer" description="building web backends and software"></div>
    <div d-option value="design" text="designer" description="crafting user interfaces"></div>
  </div>
  <div d-step name="tools" title="which tools do you use?" type="multiple" skippable>
    <div d-option value="git" text="git"></div>
    <div d-option value="docker" text="docker"></div>
  </div>
  <div d-step name="notes" title="any extra thoughts?" type="text" placeholder="write your thoughts"></div>
</d-questionnaire>
```

<a name="attributes-and-properties-d-questionnaire"></a>
## attributes and properties

| attribute        | property        | type      | default | form | description                                                   |
| :--------------- | :-------------- | :-------- | :------ | :--- | :------------------------------------------------------------ |
| `name`           | `name`          | `string`  | `""`    | no   | form field name                                               |
| `value`          | `value`         | `string`  | `"{}"`  | yes  | serialized json payload of answers (form associated)          |
| `current-step`   | `currentStep`   | `number`  | `0`     | no   | current active step index                                     |
| `auto-advance`   | `autoAdvance`   | `boolean` | `false` | no   | automatically advance on single choice selection              |
| `show-progress`  | `showProgress`  | `boolean` | `true`  | no   | displays the animated progress bar                            |
| `show-shortcuts` | `showShortcuts` | `boolean` | `true`  | no   | displays numeric key badges on options                        |
| `questions`      | `questions`     | `string`  | `""`    | no   | json string defining steps as an alternative to html children |

<a name="programmatic-api-d-questionnaire"></a>
## programmatic api

```javascript
const qn = select('d-questionnaire');

// navigation methods
qn.next();             // validates and proceeds to next step or submits
qn.prev();             // navigates to previous step
qn.skip();             // skips current step if optional or skippable
qn.goTo(1);            // jumps to specified step index
qn.reset();            // clears answers and resets to step 0

// answers access
const ans = qn.answers;              // returns copy of collected answers
qn.setAnswer('role', 'dev');        // sets answer programmatically
```

<a name="events-d-questionnaire"></a>
## events

| event    | detail                                     | bubbles | description                                  |
| :------- | :----------------------------------------- | :------ | :------------------------------------------- |
| `step`   | object (`{ step, index, total, answers }`) | yes     | emitted when transitioning to a new step     |
| `answer` | object (`{ name, value, answers }`)        | yes     | emitted whenever an answer is set or toggled |
| `submit` | object (`{ answers, json }`)               | yes     | emitted when completing the final step       |

<a name="behavior-d-questionnaire"></a>
## behavior

- supports keyboard navigation (numbers `1` through `9` select matching options, `Enter` continues, `Backspace` or `ArrowLeft` goes back).
- validates required steps and triggers a shake animation when attempting to continue without answering.
- serializes collected answers to json and integrates with standard `<form>` and `<d-form>` submissions.
