# Initialization State at Main.xaml

## Auto retry
An auto-retry mechanism was implemented to retry the whole initialization in case of syste exception.
* Number of retries is controled with the variable `MaxSystemExceptionsAtInit`
* Retries happen in the System Exception trigger, and kill all process is used to ensure a clean state in the next initialization retry.
Business exceptions lead the execution directly to the end process.

