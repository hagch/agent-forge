---
name: dau-ui-test
description: >
  Use when testing a feature from the perspective of a non-technical
  user. Browser-based UI walkthrough checking clarity, feedback, and
  edge cases. Triggers: verification phase, UX review, feature
  completion, before shipping.
---

# DAU-UI-Test

Test the feature as a non-technical, impatient user who reads nothing.

## The DAU Mindset

- You do NOT know the feature exists until you find it
- You do NOT read documentation, tooltips, or help text (at first)
- You click the most obvious thing
- You get confused easily
- You blame the software, never yourself

## Test Script

### 1. Discovery
- Start from the main page / dashboard
- Can you FIND the new feature without being told where it is?
- Is it in the navigation? Is the label clear?
- Would you know what it does from the label alone?

### 2. First Interaction
- Click into the feature
- Is it immediately clear what this page/view does?
- Is there a call-to-action? Do you know what to do first?
- If the page is empty (no data yet): is there a helpful message?

### 3. Happy Path
- Complete the main action (create, edit, submit)
- Was each step clear?
- Did you get feedback after each action? (toast, redirect, visual change)
- Did you know it worked?

### 4. Error Path
- Submit a form with missing required fields
- Are the error messages understandable? (not "400 Bad Request")
- Do errors appear near the field, not just at the top?
- After fixing errors, does submit work?

### 5. Edge Cases
- Enter very long text (500+ characters in a name field)
- Enter special characters: `<script>`, `' OR 1=1`, emojis
- Use the back button mid-flow
- Open the same page in two tabs
- Try on a narrow screen (mobile viewport)

### 6. Accessibility Quick Check
- Tab through the feature — can you reach everything?
- Are interactive elements focusable and visually indicated?
- Do images have alt text?
- Are form fields labeled?

## Documentation Format

```markdown
### DAU-UI-Test Results

#### Discovery: PASS | FAIL
- <How you found the feature, what was clear/unclear>

#### First Interaction: PASS | FAIL
- <What was immediately clear, what was not>

#### Happy Path: PASS | FAIL
- <Step-by-step journey, feedback received>

#### Error Path: PASS | FAIL
- <Error messages seen, clarity assessment>

#### Edge Cases: PASS | FAIL
- <What happened with unusual input>

#### Accessibility: PASS | FAIL
- <Keyboard navigation, focus indicators, labels>

#### Overall: PASS | FAIL
- <Summary: would a non-technical user succeed?>
```
