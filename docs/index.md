# Run `nvidia-smi` within a Slurm script 


Add the following block to your Slurm script. 

* `STATS_INTERVAL=5` : This will gather profile data every 5 seconds .i
* `STATS_FILE` : Make to include a valid path to write the .csv file which contains the profile data
* `nvidia-smi --query-gpu=` : Most essential variables are `utilization.gpu,utilization.memory,memory.used,memory.total`. Feel free to remove the remainder

!!! info ""

    ```
    # GPU usage monitoring
    STATS_INTERVAL=5
    STATS_FILE="/path/to/save/the/output/file/${SLURM_JOB_ID}-gpu_stats.csv"
    nvidia-smi --query-gpu=timestamp,uuid,clocks_throttle_reasons.sw_thermal_slowdown,utilization.gpu,utilization.memory,memory.used,memory.total,temperature.gpu,power.draw,clocks.current.sm \
        --format=csv,nounits -l "$STATS_INTERVAL" -f "$STATS_FILE" &
    sleep 20
    ```

### Visualising Profile data

```
#replace 18957616 with the corresponding JobID

import pandas as pd
dset = pd.read_csv("18957616-gpu_stats.csv", parse_dates=["timestamp"], index_col="timestamp", sep=", ", engine="python")
dset = dset.drop(columns="memory.total [MiB]").dropna()
dset["sw_thermal_slowdown"] = dset["clocks_throttle_reasons.sw_thermal_slowdown"].apply(lambda x: 0 if "Not" in x else 1)
axes = dset.plot(subplots=True, figsize=(15, 10), grid=True)
axes[0].figure.savefig("gpu_profile_figure.png")
```