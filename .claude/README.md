# Claude Skills & Expertise System

This directory contains the **Expert Skill Creation System** for FigmaToCode, demonstrating how to transform comprehensive documentation into intelligent, progressive-disclosure-based assistants.

## Contents

### 📚 Expertise Files

1. **`expertise-index.json`**
   - Structured index of all FigmaToCode expertise
   - Maps topics, queries, and workflows
   - Defines progressive disclosure levels
   - **Purpose**: Routes queries to the right documentation at the right depth

2. **`figmatocode-expert.md`**
   - Expert skill prompt for FigmaToCode assistance
   - Uses progressive disclosure (4 levels of detail)
   - Provides context-aware help
   - **Purpose**: The actual "expert" that uses the index

3. **`skill-creator-system.md`**
   - Complete system documentation
   - Template for creating expert skills from any documentation
   - Best practices and guidelines
   - **Purpose**: Meta-documentation on how to create expert skills

## How It Works

### The Two-Part System

```
┌─────────────────────────────────────────────────────────┐
│  PART 1: Expertise Extractor                            │
│  (What we did in /report folder)                        │
│                                                          │
│  Input: Codebase                                        │
│  Output: Comprehensive documentation                    │
│                                                          │
│  Files:                                                 │
│  - report/architecture.md                               │
│  - report/capabilities.md                               │
│  - report/frameworks.md                                 │
│  - etc. (11 documentation files)                        │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│  PART 2: Expert Skill Creator                           │
│  (What's in this .claude folder)                        │
│                                                          │
│  Input: Documentation                                   │
│  Output: Expert skill with progressive disclosure       │
│                                                          │
│  Files:                                                 │
│  - expertise-index.json (index/router)                  │
│  - figmatocode-expert.md (expert skill)                 │
│  - skill-creator-system.md (how to create more)         │
└─────────────────────────────────────────────────────────┘
```

### Progressive Disclosure

The expert skill uses **4 levels** of information disclosure:

**Level 1: Quick Answer** (2-3 sentences + link)
```
User: "What is FigmaToCode?"
Expert: Brief description + link to docs
```

**Level 2: Summary** (key points + examples)
```
User: "How does Tailwind generation work?"
Expert: Key features + code example + links
```

**Level 3: Detailed** (full section with examples)
```
User: "Why does rotation not work correctly?"
Expert: Complete algorithm explanation + workarounds
```

**Level 4: Deep Dive** (multiple related topics)
```
User: "Help me debug color conversion issues"
Expert: Conversion pipeline + algorithm + settings + limitations
```

## Usage

### Using the FigmaToCode Expert

**In an AI assistant or agent:**

1. Load the expert skill:
   ```
   Read: .claude/figmatocode-expert.md
   ```

2. The expert will:
   - Check the expertise index
   - Route your query to relevant topics
   - Provide answers at appropriate depth
   - Offer to expand if needed

3. Examples:
   ```
   "What frameworks are supported?"
   → Level 1 response with links

   "How to add a new framework?"
   → Level 2 summary with steps

   "Debug: colors not matching Figma"
   → Level 3 detailed explanation + workarounds
   ```

### Creating Your Own Expert Skills

**To create an expert for any documentation:**

1. **Extract Expertise** (separate process)
   - Analyze the codebase/system
   - Create comprehensive documentation
   - Organize into logical topics

2. **Create Index** (use template in skill-creator-system.md)
   ```json
   {
     "expertise_name": "Your Expert Name",
     "topics": { ... },
     "query_patterns": { ... },
     "workflows": { ... }
   }
   ```

3. **Generate Skill** (use template in skill-creator-system.md)
   ```markdown
   # Your Expert Skill

   Progressive disclosure expert on [topic].
   [Follow the template structure]
   ```

4. **Test**
   - Try queries at different levels
   - Verify progressive disclosure works
   - Ensure routing is correct

See **`skill-creator-system.md`** for complete instructions.

## File Relationships

```
expertise-index.json
    ↓ (referenced by)
figmatocode-expert.md
    ↓ (reads and routes to)
/report/*.md (documentation files)
    ↓ (created by following)
skill-creator-system.md (template/guide)
```

## Key Concepts

### 1. Progressive Disclosure

**Problem**: Dumping all information overwhelms users

**Solution**: Start minimal, expand as needed
- Level 1: Quick answer
- Level 2: Summary
- Level 3: Detailed
- Level 4: Deep dive

**Benefit**: Users get exactly what they need, no more, no less

### 2. Query Routing

**Problem**: How to find the right information quickly?

**Solution**: Map queries to topics
- "What is..." → overview
- "How to..." → getting-started
- "Why does..." → algorithms
- "Can it..." → capabilities

**Benefit**: Fast, accurate information retrieval

### 3. Workflow Guidance

**Problem**: Users have different goals

**Solution**: Define common workflows
- Evaluating tool → overview → capabilities → limitations
- Contributing → getting-started → architecture
- Debugging → conversion-pipeline → algorithms

**Benefit**: Contextual, goal-oriented assistance

## Examples

### Example 1: Quick Question

**User**: "What is FigmaToCode?"

**Expert** (Level 1):
```markdown
**Quick Answer:**

FigmaToCode is a Figma plugin that converts designs into working code
for HTML, Tailwind, Flutter, SwiftUI, and Jetpack Compose using
deterministic algorithms (no AI).

📚 **Learn More:** [Overview](../report/README.md)
```

### Example 2: How-To Question

**User**: "How to configure Tailwind prefix?"

**Expert** (Level 2):
```markdown
**Tailwind Prefix Setting**

Use the `customTailwindPrefix` setting to add a prefix to all classes:

**Configuration:**
```json
{
  "customTailwindPrefix": "tw-"
}
```

**Output:**
```html
<div class="tw-flex tw-gap-4 tw-bg-blue-500">
```

⚙️ **Full Settings:** [Plugin Settings](../report/plugin-settings.md#custom-tailwind-prefix)

Want to know about other Tailwind settings?
```

### Example 3: Debugging Question

**User**: "Colors aren't converting correctly"

**Expert** (Level 3):
```markdown
**Color Conversion Debugging**

[Detailed explanation of color conversion algorithm]
[Common issues and causes]
[Settings that affect color conversion]
[Workarounds and solutions]

📚 **Algorithm Details:** [Color Conversion](../report/algorithms.md#color-conversion)
⚙️ **Settings:** [Plugin Settings](../report/plugin-settings.md#round-tailwind-colors)
💡 **Limitations:** [Known Issues](../report/limitations.md)

Would you like help with a specific color issue?
```

## Benefits

### For Users

✅ **No Information Overload**: Get exactly what you need
✅ **Fast Answers**: Quick responses for simple questions
✅ **Deep Help Available**: Can always go deeper
✅ **Contextual**: Understands your goal
✅ **Self-Service**: Documentation always linked

### For Maintainers

✅ **Reusable**: Create expert for any documentation
✅ **Scalable**: Works with large documentation sets
✅ **Maintainable**: Index separates routing from content
✅ **Consistent**: Standard format across topics
✅ **Extensible**: Easy to add new topics/workflows

## Integration

### With AI Assistants

Load the expert skill in your AI assistant:

```
Read: .claude/figmatocode-expert.md

Then ask questions about FigmaToCode
```

The assistant will use progressive disclosure automatically.

### With Documentation

The expert doesn't replace documentation:
- Documentation = comprehensive reference
- Expert = intelligent navigation layer
- Both complement each other

### With Development

Developers can:
1. Update documentation in `/report`
2. Update index in `expertise-index.json`
3. Expert skill automatically uses new content

No need to retrain or rebuild the expert!

## Future Enhancements

Potential additions:

1. **Interactive Tutorials**: Step-by-step guided learning
2. **Context Awareness**: Remember previous questions
3. **Difficulty Adaptation**: Adjust depth based on user level
4. **Multi-Expert**: Multiple specialized experts
5. **Automated Index**: Generate index from documentation
6. **Version Support**: Handle multiple doc versions

## Summary

This `.claude` directory demonstrates a complete system for creating intelligent expert assistants from documentation:

1. **Extract expertise** → Comprehensive docs in `/report`
2. **Index expertise** → `expertise-index.json`
3. **Create expert** → `figmatocode-expert.md`
4. **Use progressive disclosure** → Right info, right depth, right time

**Result**: Users get helpful, non-overwhelming assistance from comprehensive documentation.

---

For more details on creating your own expert skills, see **`skill-creator-system.md`**.
