---
name: skill-creator
description: Create new skills for the Abacus AI Agent with proper structure, metadata, and best practices. All skills should be written in English to ensure consistency and maintainability across the codebase.
metadata:
  author: Miguel Angelo
  license: MIT
  version: 1.0.0
---

# Skill Creator

## Purpose

Create new skills for the Abacus AI Agent following proper structure, metadata standards, and best practices. This skill ensures consistency across all skills in the workspace.

## Language Requirement

**CRITICAL**: All skills must be written in **English**.

- Write all skill descriptions, instructions, and documentation in English
- Use English for code comments and examples within skills
- Maintain English as the primary language for all skill-related content
- Only user-facing output should be in Portuguese (or the user's selected language)
- This ensures consistency, maintainability, and easier collaboration across the codebase

## Skill Structure

A valid skill MUST follow this directory structure:

```
.abacusai/skills/<skill-name>/
├── SKILL.md              # Main skill definition (REQUIRED)
├── README.md             # Optional: Extended documentation
├── references/           # Optional: Reference documents
│   ├── example1.md
│   └── example2.md
└── scripts/              # Optional: Helper scripts
    └── script.sh
```

## SKILL.md Format

The SKILL.md file must follow this exact format:

### Frontmatter (REQUIRED)

```yaml
---
name: skill-name
description: Single sentence description of what the skill does
metadata:
  author: Author Name
  license: MIT (or appropriate license)
  version: X.Y.Z
---
```

**Requirements**:

- `name`: kebab-case, no spaces, unique across all skills
- `description`: concise single sentence
- `author`: your name or organization name
- `license`: use MIT for compatibility
- `version`: semantic versioning (major.minor.patch)

### Content Sections (REQUIRED)

After the frontmatter, include these sections:

#### Purpose

Brief paragraph explaining what the skill does and when to use it.

#### Instructions

Step-by-step instructions for how the skill should be used. Be specific and actionable.

#### Best Practices (if applicable)

List of best practices relevant to this skill.

#### Examples (if applicable)

Concrete examples showing how to use the skill.

#### Triggers (if applicable)

List of keywords or phrases that should trigger this skill.

#### Notes/Constraints (if applicable)

Important limitations or special considerations.

## Skill Creation Process

When creating a new skill:

1. **Determine the purpose**: Clearly define what problem the skill solves
2. **Choose a descriptive name**: Use kebab-case, be specific
3. **Write comprehensive instructions**: Include clear, actionable steps
4. **Add examples**: Show concrete usage patterns
5. **Define triggers**: List keywords that should activate this skill
6. **Test the skill**: Verify it works as expected
7. **Document dependencies**: List any required tools or scripts

## Best Practices for All Skills

### General Guidelines

- **Keep it focused**: Each skill should do one thing well
- **Be explicit**: Avoid ambiguous instructions
- **Use imperative mood**: Tell the agent what to do directly
- **Include examples**: Show, don't just tell
- **Define boundaries**: Clearly state what the skill does NOT do
- **Version properly**: Update version numbers for meaningful changes

### Writing Instructions

- Use numbered lists for sequential steps
- Use bullet points for non-sequential items
- Include edge cases and error handling
- Specify output format requirements
- Define success criteria

### Metadata Standards

- Always include the full metadata section
- Use semantic versioning (major.minor.patch)
- Update version numbers for:
  - **Major**: Breaking changes or complete rewrites
  - **Minor**: New features or significant improvements
  - **Patch**: Bug fixes or minor improvements
- Include author contact information when relevant

### Documentation

- Keep descriptions concise but complete
- Use clear section headers
- Provide context before technical details
- Include troubleshooting tips when applicable
- Document all optional parameters and their defaults

## Templates

### Basic Skill Template

```markdown
---
name: your-skill-name
description: Single sentence description of what this skill does
metadata:
  author: Your Name
  license: MIT
  version: 1.0.0
---

# Skill Name

## Purpose

Brief paragraph explaining what this skill does and when to use it.

## Instructions

1. Step one
2. Step two
3. Step three

## Best Practices

- Best practice one
- Best practice two

## Examples

Example 1: [brief description]

[Show what happens]

Example 2: [brief description]

[Show what happens]

## Notes

Any important limitations or special considerations.
```

### Advanced Skill Template (with scripts)

```markdown
---
name: advanced-skill
description: A skill with helper scripts and references
metadata:
  author: Your Name
  license: MIT
  version: 1.0.0
---

# Advanced Skill

## Purpose

[Description]

## Instructions

[Instructions]

## Scripts

This skill includes the following helper scripts:

- `scripts/script-name.sh`: [what it does]

Usage: `node .abacusai/skills/advanced-skill/scripts/script-name.sh [args]`

## References

- `references/guide.md`: Additional documentation
- `references/examples.md`: Code examples
```

## Skill Registration

After creating a skill:

1. Place it in `.abacusai/skills/<skill-name>/`
2. Create the SKILL.md with proper frontmatter
3. Add any supporting files (scripts, references)
4. Test that the skill is recognized by the system
5. Document any special requirements

## Common Mistakes to Avoid

- **Missing frontmatter**: Always include the YAML frontmatter
- **Inconsistent naming**: Use the same name in directory and frontmatter
- **Vague descriptions**: Be specific about what the skill does
- **Missing examples**: Examples make skills easier to understand
- **Ignoring edge cases**: Consider what happens when things go wrong
- **Overlapping skills**: Skills should have clear, distinct purposes
- **Non-English content**: All skill content must be in English

## Troubleshooting

### Skill not recognized

- Check that SKILL.md exists in the skill directory
- Verify frontmatter is valid YAML
- Ensure the name matches the directory name

### Instructions not followed

- Review instructions for clarity and completeness
- Add more examples if needed
- Check that triggers are appropriate

### Version conflicts

- Use semantic versioning
- Update version numbers for all changes
- Document breaking changes in the skill description

## Maintenance

Keep skills up to date by:

- Regularly reviewing instructions for accuracy
- Updating examples as the codebase evolves
- Fixing bugs and improving clarity
- Incrementing version numbers appropriately
- Removing deprecated features

## Related Skills

List related skills with brief descriptions:

- `git-commit`: Generate conventional commit messages
- `onp-spec-driven`: Spec-anchored development workflow
- `testing`: Ensure code quality through automated testing

## Contributing

When contributing new skills:

1. Follow this template exactly
2. Write everything in English
3. Include comprehensive examples
4. Test thoroughly before submitting
5. Update this skill-creator documentation if patterns change

---