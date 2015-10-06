The flows to be exported from contrail-vrouter-agent to contrail-collector are selected from a algorithm. The parameters for the algorithm are
    (1) configured flow export rate 
    (2) actual flow export rate and 
    (3) sampling threshold.

Each flow which needs to be exported is subjected to the following algorithm to decide whether it needs to exported or dropped.
    
        (1) Flow samples greater than or equal to sampling threshold will always be
        exported, with the byte/packet counts reported as-is.
        (2) Flow samples smaller than the sampling threshold will be exported
         probabilistically, with the byte/packets counts adjusted upwards according to
         the probability.
        (3) Probability =  diff_bytes/sampling_threshold
        (4) We generate a random number less than sampling threshold.
        (5) If the diff_bytes is less than the random number then the flow is dropped.
        (6) Otherwise the flow is exported after normalizing the diff bytes and
         packets. The normalization is done by dividing diff_bytes and diff_pkts with
         probability. This normalization is used as heuristictic to account for stats
         of dropped flows
    
The actual flow-export-rate will be close to the configured configured export rate and whenever there is huge deviations we adjust sampling threshold to bring the actual flow export rate close to configured flow export rate. It is not guaranteed that the actual flow export rate will always be close to configured flow export rate.
