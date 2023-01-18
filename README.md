# Predicer
‘Predictive decider’ for actors making decisions over multiple stages

## How to install

- Clone the project to your computer using git.
- Make sure that the main branch is activated.
- Open a julia REPL. It is recommended to use a IDE such as VSCode or Atom, as the results from the model are currently not saved in a separate file.
- Navigate to *.\Predicer* using the `cd` command.

        julia> cd("./Predicer)

- type `]` to open the package manager in julia, and type `activate .` to activate the local Julia environment.

        (Predicer) pkg> activate .

- Press backspace to exit the package manager
- Type `using Pkg` followed by `Pkg.instantiate()` to install the required packages. This can take a while.

        julia> using Pkg
        julia> Pkg.instantiate()

- Finally type `using Predicer` to use the package.

## How to use

- Navigate to the local folder containing the Predicer package.
- type `]` to open the package manager in julia, and type `activate .` to activate the local Julia environment.

        (Predicer) pkg> activate .

- Press backspace to exit the package manager
- Type `using Predicer` to use the package.

        julia> using Predicer

- To generate a model based on a input data file (in the example an Excel file located under `Predicer\\input_data\\`) use the `Predicer.generate_model(fpath)` function, where the parameter 'fpath' is the path to the input data file. The 'generate_model()' function imports the input data from the defined location, and build a model around it. The function returns two values, a "model contents" (mc) dictionary containing the built optimization model, as well as used expressions, indices and constraints for debugging. The other return value is the input data on which the optimization model is built on. 
        
        julia> mc, input_data = Predicer.generate_model(fpath)

- Or if using the example input data file `Predicer\\input_data\\input_data.xlsx`

        julia> mc, input_data = Predicer.generate_model(joinpath(pwd(), "input_data\\input_data.xlsx"))


- `Predicer.solve_model(mc)` solves the model `mc`, and shows the solver output.

        julia> Predicer.solve_model(mc)


- The resulting bid matrix can be exported to a .xlsx file under `Predicer\\results` by using the `Predicer.write_bid_matrix()` function

        julia> Predicer.write_bid_matrix(mc, input_data)


## Input data description

The basic parameters and usage of the excel-format input data are described here. The input data has to be given to Predicer in a specific form, and the excel-format input data files are not a requirement. Excel has been used during development, since they were considered more convenient than databases or other forms of data structures.

### Node

Nodes are fundamental building blocks in Predicer, along with Processes.

| Parameter               | Type   | Description                                                |
|-------------------------|--------|------------------------------------------------------------|
| node                    | String | Name of the node                                           |
| is_commodity            | Bool   | Indicates if the node is a commodity node                  |
| is_state                | Bool   | Indicates if the node has a state (storage)                |
| is_res                  | Bool   | Indicates if the node is involved in reserve markets       |
| is_market               | Bool   | Indicates if the node is a market node                     |
| is_inflow               | Bool   | Indicates if the node has an inflow                        |
| state_max               | Float  | Storage state capacity (if node has a state)               |
| in_max                  | Float  | Storage state charge capacity                              |
| out_max                 | Float  | Storage state discharge capacity                           |
| initial_state           | Float  | Initial state of the storage                               |
| state_loss_proportional | Float  | Hourly storage loss relative to the state of the storage   |
| residual_value          | Float  | Value of the storage contents at the end of the time range |


### Processes

Processes are fundamental building blocks in Predicer, along with Nodes. They are used to convert or transfer electricity or heat, or etc. between nodes in the modelled system.

| Parameter     | Type    | Description                                                                                         |
|---------------|---------|-----------------------------------------------------------------------------------------------------|
| process       | String  | Name of the process                                                                                 |
| is_cf         | Bool    | Indicates if the process is limited by a capacity factor time series                                |
| is_cf_fix     | Bool    | Indicates if the process has to match the capacity factor time series                               |
| is_online     | Bool    | Indicates if the process is an online/offline unit                                                  |
| is_res        | Bool    | Indicates if the process participates in reserve markets                                            |
| conversion    | Integer | Indicates the type of the process. 1 = unit based process, 2 = transfer process, 3 = market process |
| eff           | Float   | Process efficiency (total output / total input)                                                     |
| load_min      | Float   | Minimum load of the process as a fraction of total capacity. Only for online processes              |
| load_max      | Float   | Maximum load of the process as a fraction of total capacity. Only for online processes              |
| start_cost    | Float   | Cost of starting the unit, only for online processes.                                               |
| min_online    | Float   | Minimum time the process has to be online after start up                                            |
| min_offline   | Float   | Minimum time the process has to be offline during shut down                                         |
| max_online    | Float   | Maximum time the process can be online                                                              |
| max_offline   | Float   | Maximum time the process can be offline                                                             |
| initial_state | Bool    | Initial state of the online unit (0 = offline, 1 = online)                                          |


### Process topology

Process topologies are used to define the process flows and capacities in the modelled system. Flows are connections between nodes and processes, and are used to balance the modelled system.

| Parameter   | Type   | Description                                                                       |
|-------------|--------|-----------------------------------------------------------------------------------|
| process     | String | Name of the process                                                               |
| source_sink | String | Determines whether the connection node is a source or a sink for the process      |
| node        | String | Name of the connection node                                                       |
| capacity    | Float  | Capacity of the connection                                                        |
| VOM_cost    | Float  | Variable operational and maintenance cost of using the corresponding process flow |
| ramp_up     | Float  | Determines the hourly upward ramp rate of the corresponding process flow          |
| ramp_down   | Float  | Determines the hourly downward ramp rate of the corresponding process flow        |


### Reserve type

The sheet 'reserve_type' is used to define the types of reserve used in the model, mainly differing based on reserve activation speed. 

| Parameter | Type | Description |
|-------------|--------|-------------------------------------------------------------------------------------------------------------------------------------------|
| type        | String | Name of the reserve type                                                                                                                  |
| ramp_factor | Float  | Ramp rate factor of reserve activation speed. (If reserve has to activate in 1 hour, ramp_factor is 1.0. In 15 minutes, ramp_factor is 4) |


### Market

Markets are a type of node, with which the modelled system can be balanced by buying or selling of a product such as electricity. Markets can either be of the *energy* type, or of the *reserve* type. 

| Parameter    | Type   | Description                                                                      |
|--------------|--------|----------------------------------------------------------------------------------|
| market       | String | Name of the market                                                               |
| type         | String | type of the market (energy or reserve)                                           |
| node         | String | Node the market is connected to                                                  |
| direction    | String | Direction of the market, only for reserve markets                                |
| realisation  | Float  | Determines the fraction of offered reserve product that activates each time step |
| reserve_type | String | Determines the type of the reserve                                               |
| is_bid       | Bool   | Determines if bids can be offered to the market                                  |


### Time series data

Time series are used in Predicer to represent parameters that are time-dependent. The notation to define time series data in the excel input files depend on the time series data in question. 

The sheet *timeseries* contains the timesteps used in the model. This sheet contains only one column *t*, with the given time steps.

#### Time series notation in the excel-format input data

The first column, *t*, should contain the time steps for the time series data. The following columns (2-n) should contain the values corresponding to the given time steps. The name of the other columns should start with the name of the linked entity, usually followed by which scenario the value is for. The values can be defined for each scenario in separate columns, or a single column can be used for several scenarios, separated by commas. The notation used for the different time series is given in the table below.

As an example the inflow for the node *nn* can be given as *nn,s1* if the values are given for scenario *s1*, and *nn,s1,s2* if the given values should be for both *s1* and *s2*. If all scenarios should have the same values, they can be defined as *nn,ALL*. 

| Sheet          | Description                                                                 | Notation                          |
|----------------|-----------------------------------------------------------------------------|-----------------------------------|
| cf             | Capacity factor time series for processes with cf functionality             | process, scenario(s)              |
| inflow         | Inflow time series for nodes (inflow positive value, demand negative value) | node, scenario(s)                 |
| price          | Price time series for the cost of using commodity nodes                     | node, scenario(s)                 |
| market_prices  | Price time series for the defined markets                                   | market, scenario(s)               |
| balance_prices | Price time series for balance markets                                       | market, direction, scenario(s)    |
| fixed_ts       | Value time series for setting market volumes to a fixed value               | market                            |
| eff_ts         | Value time series of the efficiency of processes                            | process, scenariox(s)             |
| cap_ts         | Value time series limiting a flow of a process                              | Process, connected node, scenario |


### Scenario

The scenarios in Predicer are separate versions of the future, with potentially differing parameter values. Predicer optimizes the optimal course of action, based on the probability of the defined scenarios.

| Parameter   | Type   | Description                                                  |
|-------------|--------|--------------------------------------------------------------|
| name        | String | Name of the scenario                                         |
| probability | Float  | Probability of the scenario. The sum of all rows should be 1 |

### Risk

The *risk* sheet in the excel-format input data contains information about the CVaR (conditional value at risk). For details, see [[1]](#1) and [[2]](#2)


| Risk parameter | Description                         |
|----------------|-------------------------------------|
| alfa           | Risk quantile                       |
| beta           | Share of CVaR in objective function |

### General constraints

General constraints in Predicer can be used to limit or fix process flows in relation to other process flows or a fixed value. The name and type of the constraint is defined in the sheet *constraints*, and the factors are defined in the sheet *gen_constraint*. 

#### constraint

| Parameter | Type   | Description                                                                              |
|-----------|--------|------------------------------------------------------------------------------------------|
| name      | String | Name of the general constraint                                                           |
| operator  | String | The operator used in the general constraint. *eq* for *=*, *gt* for *>* and *lt* for *<* |

#### gen_constraint

The time series data in the sheet *gen_constraint* has a special notation. As with other time series data, the first column *t* in the sheet *gen_constraint* contains the time steps for the constraint. The rest of the columns (2-n) contain information corresponding to specific constraints, defined in the sheet *constraint*. General constraints contain factors, which add up to a constant. The notation for the first row of columns (2-n) is *constraint name, process, process flow, scenario* for factors and *constraint name, scenario* for the constant.

As an example, assume a CHP gas turbine *gt* with a input flow of natural gas *ng*, as well as outputs of electricity *elc* and heat in the form of exhaust gases *heat*. Assume, that the ratio of heat and power should be 3:1, meaning an electrical efficiency of 25%, and the remaining 75% being heat. To ensure this ratio, a general constraint named *gt_c1* is defined in the sheet *constraint*, with the operator set to *eq*. 



| name  | operator |
|-------|----------|
| gt_c1 | eq       |

In the sheet *gen_constraint*, the names of the columns referencing to the factors should be *gt_c1,gt,heat,s1* and *gt_c1,gt,elc,s1*. This refers to the constraint *gt_c1*, the process *gt*, and the process flows *heat* and *elc*. The name of the column referencing to the constant should be *gt_c1,s1*. As the sum of the factors equal the constant, the value of the factors should be *3* or *-3* for the factor representing the electricity flow, and *-1* or *1*, respectively, for the factor representing the heat flow. The value of the constant should be *0*. As the sum of the factors equal the constant, it would lead to *3 \* elc - 1 \* heat = 0* or alternatively *-3 \* elc + 1 \* heat = 0*.

| t  | gt_c1,gt,elc,s1 | gt_c1,gt,heat,s1 | gt_c1,s1|
|----|-----------------|------------------|---------|
| t1 | 3               | -1               | 0       |
| t2 | 3               | -1               | 0       | 
| tn | 3               | -1               | 0       |

## References

<a id="1">[1]</a> 
Krokhmal, P., Uryasev, S., and Palmquist, J., “Portfolio optimization with conditional value-at-risk objective and constraints,” J. Risk, vol. 4, no. 2, pp. 43–68, 2001, doi: 10.21314/jor.2002.057.

<a id="2">[2]</a> 
Fleten, S. E. and Kristoffersen, T. K., “Stochastic programming for optimizing bidding strategies of a Nordic hydropower producer,” Eur. J. Oper. Res., vol. 181, no. 2, pp. 916–928, 2007, doi: 10.1016/j.ejor.2006.08.023.