# PackeRL: Dec'24 ns-3 TCP TxBuffer Bug Report 

### <span style="color:red">**Important: This is a custom fork for ns-3 bug reporting purposes! For the code of our 2024 TMLR paper, check out https://github.com/SAP/packerl.**</span>

*PackeRL* is an RL training and evaluation framework for routing optimization in computer networks. 
It leverages the [ns-3 network simulator](https://www.nsnam.org) for packet-level simulation dynamics. 

PackeRL uses its `scenarios` module (called _**Synnet**_ in the [PackeRL paper](https://openreview.net/pdf?id=H95g8UpYKY)) to generate network scenarios consisting of:

- A network topology (graph and configuration details like link capacities and delays),
- A list of traffic demands consisting of demand arrival time, volume and protocol (e.g. TCP or UDP), and
- [optional] A list of link failure events, each consisting of a failure time and the specific link that fails.

## The Bug

When simulating TCP traffic via `BulkSendApplications` with the SACK option activated, in some situations caused by our specific network scenario generation (topology + traffic demands), the TCP Scoreboard update in `TcpTxBuffer::Update()` gets called with an empty `m_sentList`. 

I do not know enough about TCP to pin down the exact issue. Yet, given the `NS_ASSERT` statements at the end of the function, I assume that something is actually broken in ns-3, and that my (very) specifc scenario generation triggers this bug.

To this end, I have created an ns-3 branch `packerl_tcp_sack_bug` where I've added an assertion in the `TcpTxBuffer::Update()` function to check for the empty `m_sentList`. Otherwise, only a minor adjustment has been made to make a `Node`'s application list protected for my custom routing logic, but that should not interfere with the TCP logic. Since this ns-3 branch is based on the current `master` branch of the ns-3 upstream, I assume that the bug is still present in the latest ns-3 version.

### Reproducing the bug

1. Clone the repository, checkout the `packerl_tcp_sack_bug` branch and initialize the submodules.
2. Create the conda environment that provides the required python packages, and activate it.
5. Install the framework in debug mode to enable `NS_LOG` prints.
6. Run the experiment that triggers the failed assertion.

**In Code:**

``` bash
git clone https://github.com/Flunzmas/packerl.git
git checkout packerl_tcp_sack_bug
git submodule update --init --recursive
conda env create -f environment.yaml
conda activate packerl_tcp_sack_bug
bash install.sh -d  # includes configuring and building of ns-3 in debug mode, with our custom contrib and scratch modules
python main.py config/sanity_check.yaml -o -t -e sanity_check
```

### Additional Notes

- If you've installed the framework wrongly, you can re-install it with `bash install.sh -rd`.
- If you'd like to adjust the logging level and ns-3 modules logged for investigation, you can do so in the `config/sanity_check.yaml` file under the `sanity_check` experiment (params `log_level` and `ns3_log_modules` in lines 22 and 23).
- The other params of the `sanity_check` experiment (episode length and random seed offset) are cherry-picked to trigger the bug. Feel free to adjust them, but the bug might take longer to appear or not appear at all.