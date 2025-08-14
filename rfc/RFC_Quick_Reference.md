# Beckn Protocol RFC Quick Reference

**Version:** 1.0  
**Date:** August 2025  

## RFC Categories

| Category | Purpose | Template | Use When |
|----------|---------|----------|----------|
| **Standards Track** | Protocol specifications | `standards-track-template.md` | New APIs, data models, protocol changes |
| **Informational** | Implementation guidance | `informational-template.md` | Best practices, case studies, guides |
| **Experimental** | Research proposals | `experimental-template.md` | Proof of concepts, novel approaches |
| **Policy** | Governance frameworks | `policy-template.md` | Security policies, compliance, governance |

## RFC Structure

### Header Section
```markdown
# RFC: [Title]

**RFC Number:** [BECKN-XXXX]  
**Category:** [Category]  
**Status:** [Draft/Proposed/Active/Deprecated]  
**Version:** [X.Y.Z]  
**Date:** [YYYY-MM-DD]  
**Updated:** [YYYY-MM-DD]  
**Authors:** [Author Names]  
**Reviewers:** [Reviewer Names]  
**Keywords:** [Comma-separated keywords]  

## Copyright Notice
[Legal text as per RFC 7841/5378]

## Status of This Memo

## Abstract
[One paragraph summary]

## Table of Contents
[Auto-generated or manual]
```

### Core Sections (All Categories)
1. **Introduction** - Background, problem statement, goals
2. **Conventions and Terminology** - Keyword interpretation as per RFC 2119/8174
3. **Technical Content** - Specifications, guidelines, or research
4. **Implementation** - How to implement or apply
5. **IANA Considerations** - IANA actions or statement of no actions
6. **Examples** - Working examples and implementations
7. **References** - Normative and informative references
8. **Appendix** - Change log, acknowledgments, glossary

## Development Process

### Phase 1: Planning (1-2 weeks)
- [ ] Define problem clearly
- [ ] Research existing solutions
- [ ] Engage stakeholders
- [ ] Choose RFC category

### Phase 2: Draft Creation (2-4 weeks)
- [ ] Select appropriate template
- [ ] Fill in all sections
- [ ] Add examples and diagrams
- [ ] Self-review for completeness

### Phase 3: Review (3-6 weeks)
- [ ] Internal technical review (1-2 weeks)
- [ ] Community review (2-4 weeks)
- [ ] Final approval (1 week)

## Writing Guidelines

### Content Quality
- **Clarity**: Use clear, concise language
- **Completeness**: Address all aspects thoroughly
- **Consistency**: Use consistent terminology
- **Precision**: Be specific, avoid ambiguity

### Technical Writing
- **Structure**: Follow template structure
- **Examples**: Provide complete, runnable examples
- **Diagrams**: Use standard notation (Mermaid, PlantUML)
- **References**: Include version information

### Documentation Standards
- **Links**: Use relative links for internal references
- **Versioning**: Use semantic versioning (X.Y.Z)
- **Change Tracking**: Maintain detailed change logs

## Review Checklist

### Content Review
- [ ] Abstract clearly summarizes the RFC
- [ ] Problem statement is well-defined
- [ ] Goals and non-goals are clear
- [ ] Technical specification is complete
- [ ] Examples are provided and correct
- [ ] Security considerations are addressed
- [ ] Backward compatibility is considered
- [ ] Implementation guidelines are clear

### Format Review
- [ ] Template is correctly used
- [ ] Header information is complete
- [ ] Table of contents is accurate
- [ ] Formatting is consistent
- [ ] Links are valid
- [ ] Code examples are properly formatted

## Common Pitfalls

### Content Issues
- ❌ Unclear problem statement
- ❌ Insufficient technical detail
- ❌ Missing use cases
- ❌ Incomplete examples

### Process Issues
- ❌ Insufficient review time
- ❌ Missing stakeholder input
- ❌ Poor communication

### Technical Issues
- ❌ Breaking changes without migration path
- ❌ Missing security analysis
- ❌ Not considering performance impact

## Tools and Resources

### Writing Tools
- **Markdown**: VS Code, Typora, Obsidian
- **Diagrams**: Mermaid, PlantUML, Draw.io
- **Validation**: Markdown Lint, Link Checker

### Review Tools
- **Version Control**: Git, GitHub, GitLab
- **Collaboration**: Slack, Discord, Email

### Documentation Tools
- **Static Sites**: Docusaurus, GitBook, MkDocs
- **API Docs**: Swagger UI, Redoc, Postman

## RFC Numbering

- **Format**: `BECKN-XXXX` (e.g., BECKN-0001)
- **Assignment**: Sequential numbering
- **Reservation**: Reserve numbers early

## Status Tracking

| Status | Description | Duration |
|--------|-------------|----------|
| **Draft** | Initial development | Variable |
| **Proposed** | Under review | 3-6 weeks |
| **Active** | Approved and in use | Ongoing |
| **Deprecated** | No longer recommended | Variable |
| **Obsolete** | No longer supported | N/A |

## Version Numbering

- **Major (X.0.0)**: Breaking changes
- **Minor (X.Y.0)**: New features, backward compatible
- **Patch (X.Y.Z)**: Bug fixes and clarifications

## Quick Commands

### Create New RFC
```bash
# Copy template
cp rfc/templates/[category]-template.md rfc/BECKN-XXXX-[title].md

# Update header
# Fill in content
# Add to version control
git add rfc/BECKN-XXXX-[title].md
git commit -m "Add RFC BECKN-XXXX: [Title]"
```

### Validate RFC
```bash
# Check markdown syntax
markdownlint rfc/BECKN-XXXX-[title].md

# Check links
markdown-link-check rfc/BECKN-XXXX-[title].md

# Spell check
aspell check rfc/BECKN-XXXX-[title].md
```

## Contact Information

- **Framework Issues**: [GitHub Issues](https://github.com/beckn/protocol-specifications/issues)
- **RFC Questions**: [Community Forum](https://community.beckn.org)
- **Template Updates**: [Pull Requests](https://github.com/beckn/protocol-specifications/pulls)

---

**Note**: This quick reference complements the full RFC Framework and Writing Guide. For detailed information, refer to the complete documentation. 