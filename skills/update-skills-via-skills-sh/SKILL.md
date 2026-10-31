---
name: update-skills-via-skills-sh
description: Use when installing or updating agent skills with skills.sh, avoiding repo-local .agents installs, or enabling OMP discovery. Not for skill authoring.
---

# Install or update skills via skills.sh

1. Record the original directory, then set the command working directory to the user's home directory. **The current working directory determines where the CLI can create `./.agents/skills/`; running from a repository can pollute that repository instead of producing the intended global installation.**

   ```bash
   project_dir=$PWD
   cd "$HOME"
   ```

2. Install or update from a source identified as `<owner>/<repo>`:

   ```bash
   DISABLE_TELEMETRY=1 npx -y skills add <owner>/<repo>
   ```

   Use the same command for both operations. Rerun it to overwrite previously installed skills with the source's current versions.

3. Verify each expected skill at the global Claude Code location; do not treat CLI success or an `.agents` directory alone as proof of installation:

   ```bash
   test -f "$HOME/.claude/skills/<skill-name>/SKILL.md"
   ```

4. Inspect the original directory for an unintended repository-local installation:

   ```bash
   if [ -d "$project_dir/.agents" ]; then
     printf 'Inspect installation output: %s\n' "$project_dir/.agents"
   fi
   ```

   If project-local installation was not requested, distinguish new output from pre-existing content. Move only confirmed unwanted output to a trash or backup location; if `.agents` pre-existed, move only the newly added skill entries. Do not recursively delete the directory.

5. Inspect the OMP configuration actually loaded in the target environment, following includes or symlinks when present rather than assuming a personal dotfiles path. Ensure `skills.customDirectories` includes `~/.claude/skills`, preserving the active configuration's existing style:

   ```yaml
   skills:
     customDirectories:
       - ~/.claude/skills
   ```

   Confirm that the configured directory exists:

   ```bash
   test -d "$HOME/.claude/skills"
   ```
