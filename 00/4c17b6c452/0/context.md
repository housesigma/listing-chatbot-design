# Session Context

## User Prompts

### Prompt 1

给这个增加一个wireframe，包含entry hero; retrieving两个主要状态

### Prompt 2

我希望这两个wireframe能够是已设计的2个状态的投影，wireframe上的元素和结构能够反应设计稿的元素和布局

### Prompt 3

wf_retrieving 的bottom sheet的圆角缺失了，调整下

### Prompt 4

在wireframe中增加组件的标注，component reference

### Prompt 5

现在的wireframe很好，我希望能够将reference在wireframe的位置和组件更贴近些

### Prompt 6

git status

### Prompt 7

commit changes

### Prompt 8

Base directory for this skill: /Users/winter/.claude/skills/git-workflow

# Git Workflow

## When to use this skill
- Creating meaningful commit messages
- Managing branches
- Merging code
- Resolving conflicts
- Collaborating with team
- Git best practices

## Instructions

### Step 1: Branch management

**Create feature branch**:
```bash
# Create and switch to new branch
git checkout -b feature/feature-name

# Or create from specific commit
git checkout -b feature/feature-name <commit-hash>...

