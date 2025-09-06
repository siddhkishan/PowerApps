# Power Apps — SharePoint Patch() Cheat Sheet

A compact, beautifully organized cheat sheet for using `Patch()` in Power Apps with SharePoint lists. Copy this file into your GitHub repo (README.md or a dedicated `.md` file) to keep as a single source of truth.

---

## Table of contents

1. Quick rules
2. Create vs Update examples
3. Field-by-field reference (common SP field types)
4. Advanced field patterns (People, Lookups, Multi-choice, Taxonomy, Attachments)
5. Helper snippets & common patterns
6. Gotchas & troubleshooting
7. How to use this file in GitHub

---

## 1) Quick rules

- **Create new item:**
  ```powerapps
  Patch(MyList, Defaults(MyList), { Field1: value1, Field2: value2 })
  ```

- **Update existing item:**
  ```powerapps
  Patch(MyList, ThisItem, { Field1: value1 })
  ```

- **Choice (single):** pass a record: `{ Value: "ChoiceLabel" }`.
- **Choice (multi):** pass a table of records: `Table({ Value: "A" }, { Value: "B" })`.
- **Lookup:** pass `{ Id: lookupId }` (Id of item in referenced list).
- **Person/Group:** pass the `Claims` value: `{ Claims: "i:0#.f|membership|user@domain.com" }` (or use Office365Users to fetch profile).

---

## 2) Create vs Update examples

### Create — simple item
```powerapps
Patch(
  MySPList,
  Defaults(MySPList),
  {
    Title: txtTitle.Text,
    Description: txtDesc.Text,
    Amount: Value(txtAmount.Text),
    DueDate: dpDueDate.SelectedDate,
    IsActive: chkActive.Value
  }
)
```

### Update — edit selected gallery item
```powerapps
Patch(
  MySPList,
  Gallery1.Selected,
  {
    Title: txtTitleEdit.Text,
    Notes: txtNotesEdit.Text,
    Tags: ForAll(ComboBoxTags.SelectedItems, { Value: ThisRecord.Value })
  }
)
```

---

## 3) Field-by-field reference

**Single line of text**
```powerapps
SingleTextInternalName: txtControl.Text
```

**Multiple lines (Plain / Rich)**
```powerapps
MultiLineField: txtMulti.Text
```

**Number**
```powerapps
NumberField: Value(txtNumber.Text)
```

**Currency**
```powerapps
CurrencyField: Value(txtCurrency.Text)
```

**Date / DateTime**
```powerapps
DateOnlyField: dpDate.SelectedDate
DateTimeField: dtDateTime.SelectedDateTime
```

**Yes / No (Boolean)**
```powerapps
BoolField: chkControl.Value
```

**Choice (single)**
```powerapps
ChoiceField: { Value: "ChoiceLabel" }
```

**Choice (multi)**
```powerapps
MultiChoiceField: Table({ Value: "A" }, { Value: "B" })
```

**Lookup (single)**
```powerapps
LookupField: { Id: 123 }
```

**Lookup (multi)**
```powerapps
LookupMulti: Table({ Id: 123 }, { Id: 456 })
```

**Person / Group (single)**
```powerapps
AssignedTo: { Claims: "i:0#.f|membership|user@domain.com" }
```

**Person / Group (multi)**
```powerapps
PeopleField: Table({ Claims: "i:0#.f|membership|user1@domain.com" }, { Claims: "i:0#.f|membership|user2@domain.com" })
```

**Hyperlink / Picture**
```powerapps
LinkField: { Value: "https://contoso.com", Description: "My site" }
ImageField: { Value: "https://..." }
```

**Attachments**
- Use the built-in **Attachments** control with `EditForm`, or upload via Power Automate after creating the item.

**Managed Metadata / Taxonomy**
- Use Power Automate for reliability, or advanced OData payloads (complex). Recommended: Flow.

---

## 4) Advanced field patterns

### Person field — get Claims from Office365Users
```powerapps
Set(u, Office365Users.UserProfileV2(txtEmail.Text));
Patch(MySPList, Defaults(MySPList), { AssignedTo: { Claims: "i:0#.f|membership|" & Lower(u.mail) } })
```

### Multi-choice from combo box
```powerapps
Patch(
  MySPList,
  Defaults(MySPList),
  {
    Tags: ForAll(ComboBoxTags.SelectedItems, { Value: ThisRecord.Value })
  }
)
```

### Lookup from combo box (Id available)
```powerapps
Patch(MySPList, Defaults(MySPList), { RelatedProject: { Id: ComboProjects.Selected.ID } })
```

### Attachments via Power Automate (pattern)
1. `Patch()` to create item; capture returned record (e.g., `Set(CreatedItem, Patch(...))`).
2. Call Flow from Power Apps: `AddAttachmentsFlow.Run(CreatedItem.ID, AttachmentControl.Attachments)`
3. Flow will iterate and upload each attachment to the created item.

### Managed Metadata — recommended approach
- Build a Flow that accepts term GUIDs or label strings and sets the taxonomy field using SharePoint connectors or HTTP requests.

---

## 5) Helper snippets & patterns

**Capture returned record after Patch**
```powerapps
Set(CreatedItem, Patch(MySPList, Defaults(MySPList), { Title: txtTitle.Text }));
// CreatedItem now holds the created record including ID
```

**Conditionally set a field only when value exists**
```powerapps
Patch(
  MySPList,
  Defaults(MySPList),
  If(IsBlank(txtOptional.Text), { Title: txtTitle.Text }, { Title: txtTitle.Text, OptionalField: txtOptional.Text })
)
```

**Patch multiple items with ForAll**
```powerapps
ForAll(
  CollToCreate,
  Patch(MySPList, Defaults(MySPList), { Title: ThisRecord.Title, Amount: ThisRecord.Amount })
)
```

---

## 6) Gotchas & troubleshooting

- **Use internal names** from List Settings — display names sometimes differ.
- **Choice values** must match configured choices exactly.
- **People fields** require tenant users; Claims string is most reliable.
- **Taxonomy & attachments** are best handled by Power Automate for consistent results.
- **Calculated & system fields** are read-only; you cannot set them.

---

## 7) How to use this file in GitHub

1. Create a new file in your repo (e.g., `docs/Patch-CheatSheet.md` or `README-Patch.md`).
2. Copy-paste this content or upload this file.
3. Optionally add to a `docs/` folder and reference from your main README.

---

## Want customization?
If you want, I can:
- Produce a **ready-to-commit** `README.md` with your list's internal names filled in (share the internal names),
- Create a **Power Automate Flow** template to upload attachments and set taxonomy fields,
- Or convert this into a **PDF** or **HTML** cheat sheet for sharing.

---

*Created for you — feel free to edit and commit to your GitHub repo.*

