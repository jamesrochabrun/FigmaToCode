# Expert Skill Creator System

This document describes the system for creating expert skills from extracted expertise documentation.

## Overview

The **Expert Skill Creator** takes comprehensive documentation (expertise) and creates an intelligent assistant skill that can:
1. Navigate the expertise efficiently
2. Use progressive disclosure to avoid overwhelming users
3. Route queries to the right documentation
4. Provide answers at appropriate depth levels

## Architecture

```
Extracted Expertise (Documentation)
         ↓
   Index Creation
         ↓
   Skill Generation
         ↓
Expert Skill with Progressive Disclosure
```

## Components

### 1. Expertise Index (JSON)

**Purpose**: Maps topics, sections, and query patterns to documentation

**Structure**:
```json
{
  "expertise_name": "Name of the Expert",
  "version": "1.0.0",
  "description": "What this expertise covers",
  "topics": {
    "topic_id": {
      "description": "What this topic covers",
      "difficulty": "beginner|intermediate|advanced",
      "file": "path/to/documentation.md",
      "sections": ["Section 1", "Section 2"],
      "related_topics": ["other_topic_id"],
      "use_when": ["query pattern 1", "query pattern 2"]
    }
  },
  "progressive_disclosure_levels": {
    "level_1_overview": {
      "description": "Quick answer",
      "max_detail": "2-3 sentences + link"
    },
    "level_2_summary": {
      "description": "Summary with examples",
      "max_detail": "Key points + code examples"
    },
    "level_3_detailed": {
      "description": "Full section",
      "max_detail": "Complete section with examples"
    },
    "level_4_deep_dive": {
      "description": "Multiple related topics",
      "max_detail": "Full context across files"
    }
  },
  "query_patterns": {
    "pattern_type": {
      "example": "Example query",
      "route_to": "topic_id",
      "disclosure_level": "level_X",
      "specific_section": "Section Name"
    }
  },
  "common_workflows": {
    "workflow_name": {
      "description": "What user is trying to do",
      "progressive_path": ["topic1 → topic2 → topic3"],
      "start_with": "topic_id",
      "disclosure_level": "recommended level"
    }
  }
}
```

### 2. Expert Skill (Markdown)

**Purpose**: Prompt template that uses the index to provide progressive disclosure

**Key Sections**:
1. **Core Principle** - Explain progressive disclosure
2. **Available Expertise Topics** - List what's available
3. **Query Routing Guide** - How to match queries to topics
4. **Response Formats** - Templates for each disclosure level
5. **Workflow Assistance** - Common user journeys
6. **Usage Instructions** - How to use the skill
7. **Examples** - Concrete examples of progressive disclosure

## Creating an Expert Skill

### Step 1: Analyze the Extracted Expertise

Review the documentation and identify:

1. **Topics**: Main subject areas covered
2. **Difficulty Levels**: Beginner, intermediate, advanced
3. **Relationships**: Which topics relate to each other
4. **Common Questions**: What users typically ask
5. **Workflows**: Common user journeys through the content

### Step 2: Create the Expertise Index

Use this template and fill in for your documentation:

```json
{
  "expertise_name": "[Your Expertise Name]",
  "version": "1.0.0",
  "description": "[What expertise covers]",
  "topics": {
    // For each major documentation file or topic:
    "[topic_id]": {
      "description": "[Topic description]",
      "difficulty": "[beginner|intermediate|advanced]",
      "file": "[path/to/file.md]",
      "sections": [
        "[Section 1]",
        "[Section 2]"
      ],
      "related_topics": ["[related_topic_id]"],
      "use_when": [
        "[query pattern that should route here]"
      ]
    }
  },
  "progressive_disclosure_levels": {
    // Standard levels - customize max_detail as needed
    "level_1_overview": {
      "description": "Quick answer with link",
      "max_detail": "2-3 sentences + link to docs"
    },
    "level_2_summary": {
      "description": "Summary of relevant section",
      "max_detail": "Key points + examples"
    },
    "level_3_detailed": {
      "description": "Full section content",
      "max_detail": "Complete section with all examples"
    },
    "level_4_deep_dive": {
      "description": "Multiple related topics",
      "max_detail": "Full context across multiple files"
    }
  },
  "query_patterns": {
    // Common query types:
    "what_is": {
      "example": "What is [X]?",
      "route_to": "overview",
      "disclosure_level": "level_1_overview"
    },
    "how_to": {
      "example": "How to [do X]?",
      "route_to": "[relevant_topic]",
      "disclosure_level": "level_2_summary"
    },
    "why_does": {
      "example": "Why does [X happen]?",
      "route_to": "[relevant_topic]",
      "disclosure_level": "level_3_detailed"
    },
    "can_it": {
      "example": "Can it [do X]?",
      "route_to": "capabilities",
      "disclosure_level": "level_2_summary"
    }
  },
  "common_workflows": {
    "[workflow_name]": {
      "description": "[What user wants to accomplish]",
      "progressive_path": ["topic1 → topic2 → topic3"],
      "start_with": "[topic_id]",
      "disclosure_level": "[recommended level]"
    }
  }
}
```

### Step 3: Create the Expert Skill Prompt

Use this template structure:

```markdown
# [Expertise Name] Expert Skill

You are a [expertise name] expert with comprehensive knowledge. Your role is to provide helpful information using **progressive disclosure**.

## Core Principle: Progressive Disclosure

[Explain the 4 levels and when to use each]

## Available Expertise Topics

[List all topics from the index with brief descriptions]

## Query Routing Guide

**Use this index**: `[path/to/expertise-index.json]`

### Common Query Patterns

[For each pattern in query_patterns, show example and routing]

## Response Format

### Level 1 (Quick Answer)
[Template for level 1 responses]

### Level 2 (Summary)
[Template for level 2 responses]

### Level 3 (Detailed)
[Template for level 3 responses]

### Level 4 (Deep Dive)
[Template for level 4 responses]

## Workflow Assistance

[For each common workflow, show the progressive path]

## How to Use This Skill

1. Understand the query
2. Check the index
3. Read only what's needed
4. Format response appropriately
5. Offer to expand

## Examples in Action

[Provide 3-5 concrete examples showing progressive disclosure]

## Important Guidelines

DO:
- Start with minimum information
- Use progressive disclosure
- Reference docs
- Offer to expand

DON'T:
- Dump entire files
- Skip documentation links
- Make up information
```

### Step 4: Test the Expert Skill

Test queries at different levels:

**Level 1 Test**: "What is [X]?"
- Should get 2-3 sentence answer + link
- No overwhelming detail

**Level 2 Test**: "How to [do Y]?"
- Should get summary with key points
- Code examples if applicable
- Link to full docs

**Level 3 Test**: "Why does [Z] not work?"
- Should get detailed explanation
- Multiple examples
- Related issues/workarounds

**Level 4 Test**: Complex debugging
- Should combine multiple topics
- Show relationships
- Comprehensive context

## Template Files

### Minimal Expert Index Template

```json
{
  "expertise_name": "",
  "version": "1.0.0",
  "description": "",
  "topics": {},
  "progressive_disclosure_levels": {
    "level_1_overview": {
      "description": "Quick answer",
      "max_detail": "2-3 sentences + link"
    },
    "level_2_summary": {
      "description": "Summary",
      "max_detail": "Key points + examples"
    },
    "level_3_detailed": {
      "description": "Detailed",
      "max_detail": "Full section"
    },
    "level_4_deep_dive": {
      "description": "Deep dive",
      "max_detail": "Multiple topics"
    }
  },
  "query_patterns": {},
  "common_workflows": {}
}
```

### Minimal Expert Skill Template

```markdown
# [Name] Expert Skill

Progressive disclosure expert on [topic].

## Core Principle

Start minimal, expand as needed.

## Topics

[List topics]

## Response Levels

1. Quick answer (2-3 sentences)
2. Summary (key points)
3. Detailed (full section)
4. Deep dive (multiple topics)

## Usage

1. Check index: `[path/to/index.json]`
2. Route query to topic
3. Choose disclosure level
4. Read only what's needed
5. Format response
6. Offer to expand
```

## Best Practices

### For Index Creation

1. **Keep topics focused**: One main concept per topic
2. **Clear descriptions**: User should know what they'll get
3. **Good routing**: Match common query patterns
4. **Link related topics**: Help users discover connections
5. **Difficulty labels**: Help gauge complexity

### For Skill Prompts

1. **Clear examples**: Show don't just tell
2. **Response templates**: Consistent formatting
3. **Concrete queries**: Real questions users ask
4. **Workflow guidance**: Common user journeys
5. **Do/Don't lists**: Clear guidelines

### For Progressive Disclosure

1. **Start small**: Level 1 is truly minimal
2. **Gradual expansion**: Each level adds more
3. **Always offer more**: Let user request depth
4. **Link documentation**: Direct access to source
5. **Cross-reference**: Show related topics

## Automation Potential

This system can be automated:

```python
# Pseudo-code for automation

def create_expert_skill(documentation_path):
    # 1. Analyze documentation structure
    topics = analyze_documentation(documentation_path)

    # 2. Generate index
    index = {
        "expertise_name": extract_name(documentation_path),
        "topics": map_topics(topics),
        "query_patterns": infer_patterns(topics),
        "workflows": identify_workflows(topics)
    }

    # 3. Generate skill prompt
    skill = generate_skill_prompt(index)

    # 4. Save files
    save_json(index, "expertise-index.json")
    save_markdown(skill, "expert-skill.md")

    return index, skill
```

## Example: FigmaToCode Expert

See the FigmaToCode expert skill as a reference implementation:
- **Index**: `.claude/expertise-index.json`
- **Skill**: `.claude/figmatocode-expert.md`

This demonstrates the complete system in action.

## Extending the System

### Adding New Features

**1. Difficulty-Based Routing**
- Route beginners to simpler explanations
- Give advanced users more technical depth

**2. Learning Paths**
- Sequential topics for learning
- Prerequisites and next steps

**3. Interactive Tutorials**
- Step-by-step walkthroughs
- Checkpoints and exercises

**4. Context Awareness**
- Remember previous queries
- Build on earlier conversations
- Suggest related topics

### Customization Points

1. **Disclosure Levels**: Adjust number and depth
2. **Query Patterns**: Add domain-specific patterns
3. **Workflows**: Define common user journeys
4. **Response Formats**: Customize templates
5. **Metadata**: Add tags, versions, etc.

## Summary

The Expert Skill Creator system:
1. Takes extracted expertise (documentation)
2. Creates an index mapping topics and queries
3. Generates a skill with progressive disclosure
4. Enables efficient, non-overwhelming assistance

**Key Benefit**: Users get exactly the information they need, when they need it, at the right level of detail.

---

*This system transforms comprehensive documentation into intelligent, progressive-disclosure-based expert assistants.*
