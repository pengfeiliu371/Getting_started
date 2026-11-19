# Claude Code skills for the group

Skills are instruction packs that [Claude Code](https://code.claude.com/docs) loads automatically when a task matches
their description. They capture workflows and cluster-specific pitfalls, so the agent does not have to rediscover them.

| Skill | What it covers |
|---|---|
| [`geoschem-pace`](geoschem-pace/SKILL.md) | Compiling and running GEOS-Chem on PACE Phoenix: GC-Classic 12.9.3 (the Nested_C1 complex-SOA setup: global 4x5 boundary conditions + 0.5x0.625 North America nest) and GCHP 14.8 (multi-node, stretched grid). Covers environments, input data, Slurm job chaining with frequent restarts, monthly QC, and a symptom → fix table. |

## Install (per user)

```bash
mkdir -p ~/.claude/skills
cp -r /path/to/Getting_started/claude-skills/geoschem-pace ~/.claude/skills/
```
Claude Code picks it up in the next session. To update, pull the repo and copy again.

The skill refers to group locations on PACE (`/storage/ideas/is-pliu40-0/ExtData`,
`/storage/project/r-pliu40-0/shared/GEOS-Chem/spack-gchp`, ...). For the human-readable tutorials, see
[Run_GCHP_on_cluster.md](../Run_GCHP_on_cluster.md) and [Install_GEOS_Chem_on_cluster.md](../Install_GEOS_Chem_on_cluster.md).
