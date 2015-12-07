contrail-vrouter-agent exports flow records to contrail-collector upon flow creation and deletion, updates flow statistics at regular intervals. Till R2.22, all flow records were exported from the agent. Depending on the scale of flows, some of these exported flows could be dropped due to queue overflow.

From R2.22, flow records are sampled and are exported to contrail-collector based on this sampling. The flows to be exported are selected based on the algorithm given below. The following parameters are used in the algorithm:

    1. Configured flow export rate (can be configured as part of global-vrouter-config)
    2. Actual flow export rate
    3. Sampling threshold (dynamic value calculated internally, if flow stats in a flow sample are above this threshold, the flow record is exported).

Each flow is subjected to the following algorithm, at regular intervals, to decide whether the sample is exported or not.
    
    1. Flows having traffic that is more than or equal to sampling threshold will always be exported, with the byte/packet counts reported as-is.
    2. Flow having traffic that is less than the sampling threshold will be exported probabilistically, with the byte/packet counts adjusted upwards according to the probability.
    3. Probability =  (bytes during the interval) / (sampling threshold)
    4. We generate a random number less than sampling threshold.
    5. If bytes during the interval is less than the random number then the flow sample is not exported.
    6. Otherwise, the flow sample is exported after normalizing the bytes during the interval and packets. Normalization is done by dividing bytes during the interval and packets during the interval with probability. This normalization is used as a heuristic to account for statistics when flow samples are dropped.

The actual flow-export-rate will be close to the configured configured export rate and whenever there is larger deviation, the sampling threshold is adjusted to bring the actual flow export rate close to configured flow export rate.