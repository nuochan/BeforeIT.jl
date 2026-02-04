# Agent Variable Documentation

> **Note**: This documentation, including the data tables and this README file, was generated with the assistance of an AI agent. While every effort has been made to ensure accuracy by verifying against the source code and live model data, users should be aware of the potential for errors or hallucinations. Please cross-reference with the original source code in `src/` for critical applications.

This folder contains structured documentation of the variables used by each agent in the BeforeIT.jl model.

## Documentation Files
All data tables are located in the `docs/agent_variable_tables` subfolder.
- `agent_variable_tables/workers_agent_transient_variables.csv`: A list of key transient variables for the `Workers` agent (shared by both active and inactive workers).
- `agent_variable_tables/workers_inactive_agent_table.csv`: Detailed breakdown of state variables for the `Workers` agent (inactive workers).
- `agent_variable_tables/workers_active_agent_table.csv`: Detailed breakdown of state variables for the `Workers` agent (active workers, including employed and unemployed).
- `agent_variable_tables/government_agent_transient_variables.csv`: A list of key transient variables for the `Government` agent.
- `agent_variable_tables/government_agent_table.csv`: Detailed breakdown of state variables for the `Government` agent.
- `agent_variable_tables/central_bank_agent_transient_variables.csv`: A list of key transient variables for the `CentralBank` agent.
- `agent_variable_tables/central_bank_agent_table.csv`: Detailed breakdown of state variables for the `CentralBank` agent.
- `docs/agent_variable_tables/bank_agent_transient_variables.csv`: A list of key transient variables for the `Bank` agent.
- `docs/agent_variable_tables/bank_agent_table.csv`: Detailed breakdown of state variables for the `Bank` agent and its embedded owner household.
- `docs/agent_variable_tables/rotw_agent_transient_variables.csv`: A list of key transient variables for the `RestOfTheWorld` agent.
- `docs/agent_variable_tables/rotw_agent_table.csv`: Detailed breakdown of state variables for the `RestOfTheWorld` agent, including CANVAS extension variables.
- `docs/agent_variable_tables/firm_agent_table.csv`: Detailed breakdown of **state variables** in the `Firms` agent role, distinguishing between the firm's production data and the owner's (investor) household data.
- `docs/agent_variable_tables/firm_agent_transient_variables.csv`: A list of key **transient (local) variables** used in firm-level calculations, which are not stored as part of the agent's state but are critical for understanding its behavior.

## Key Definitions

### Agent Role vs. Model Field
- **Agent Role**: Represents the abstract economic category of the actor (e.g., Household, Firm, Bank). This corresponds to the theoretical function of the variable.
- **Model Field**: Represents the technical location within the code (e.g., `model.firms`). 

### Scope of Transient Variable Documentation
The transient variable tables (`*_transient_variables.csv`) are not exhaustive lists of every local variable in the source code. They are curated to include only **conceptually significant variables** that act as "messengers" between different stages of an agent's decision-making process.

- **Included ("Messenger" Variables)**: Variables that are calculated in one major function and then used as an input to another, or that represent a key economic signal (e.g., `pi_c_i`, the cost-push inflation signal).
- **Excluded ("Scratchpad" Variables)**: Minor, intermediate variables used only to improve the readability of a formula within a single function (e.g., `in_sales`, `out_wages` in the `firms_profits` function). These are considered implementation details and are not critical for understanding the agent's overall logic flow.

### Data Columns
- **Variable**: The field name in the Julia struct.
- **Length**: The cardinality of the vector when initialized with the `AUSTRIA2010Q1` dataset. 
  - **Fixed**: Dimensions tied to the structure of the economy (e.g., number of firms or sectors).
  - **Non-fixed**: Dimensions tied to time/simulation period (used in CANVAS extension).
- **Path/File**: Location of the variable definition in the repository.

## Change Log
### 2026-01-30
- Created `firm_agent_table.csv` documenting 44 variables for the Firms agent.
- Identified and separated variables belonging to the **Household (Investor)** subclass embedded within `model.firms`.
- Verified vector lengths (624 for Austria) using live model initialization.
- Added summary documentation in `README.md`.
- Created `firm_agent_transient_variables.csv` to document key local variables used in firm calculations.
- Created `rotw_agent_table.csv` and verified all variable lengths and types against live model data for both standard and CANVAS versions.
- Created `rotw_agent_transient_variables.csv` to document local variables for the RoW agent.
- Created `bank_agent_table.csv` and verified all variables are scalars.
- Created `bank_agent_transient_variables.csv` to document local variables for the Bank agent.
- Re-created `central_bank_agent_table.csv` and `central_bank_agent_transient_variables.csv` as separate files for consistency.
- Moved all CSV data tables into the `docs/agent_variable_tables` subfolder for better organization.
- Created `government_agent_table.csv` and `government_agent_transient_variables.csv`, verifying variable lengths and detailing all transient revenue components.
- Corrected `central_bank_agent_transient_variables.csv` to expand folded variables into detailed rows.

### 2026-02-02
- Created `workers_active_agent_table.csv` documenting 9 state variables for the active Workers agent (4118 workers in Austria).
- Created `workers_inactive_agent_table.csv` documenting 9 state variables for the inactive Workers agent (4130 workers in Austria).
- Created `workers_agent_transient_variables.csv` documenting 2 key transient variables shared by both active and inactive workers.
- Note: Workers agent has two separate instances in the model: `model.w_act` (active workers including employed and unemployed) and `model.w_inact` (inactive workers such as retirees).
