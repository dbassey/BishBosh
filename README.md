# Verify TxM Machine Learning Tool (verify-txm-ml) - inquisitivo bear
SecOps transaction monitoring machine learning deployment scripts for Verify

# Overview
The TxM machine learning tool applies data anaylsis against TxM logs to create a model, which in turns allows us to enrich the TxM data with weights that can later be used for triaging false positives. The model is updated nightly using TxM scoring data ('scoring' being a flag set by our analysts, which trains the model).

# What's what?
Hopefully the Ansible will be fairly self explanatory, with the tool being installed by default ot /usr/local/verify-txm-ml.

Three shell scripts run the tool via cron, those being:

check_txm.sh - A script that checks that the TxM model is updating as expected, and alerts via e-mail if it's not.
TxM-Run.sh - A script that runs predict.py over the last 5 minutes of TxM logs, outputting a scored/weighted log entry. Includes some lock logic to prevent it running more than once at any time.
TxM-Run-Training.sh - A script that runs the nightly model training, utilising several underlying scripts.

# How is the weighting processed?
Currently predict.py, ran via TxM-Run.sh, outputs to two files based on the score. Anything with a score lower than 0.1 (10%) is pushed to the logroot_below file (currently /var/log/MachineID_ML_triggered_noalarm) and anything with a high score is pushed to the logroot_above file (currently /var/log/MachineID_ML_triggered). The SIEM tool of your choice can then reads these in and alarm as necessary.

The score/threshold is controlled by the following in the predict.py script:

    parser.add_argument('--threshold_score', type=float,
                        dest='threshold_score', help='threshold score - used to ignore alarms that score <= threshold.',
                        default=0.1)
Endex.

# Talk to me about Ansible
The ansible role for this has primarily been put together for Debian 8.7, but should work on other flavours.

To deploy, simply add a valid inventory and run:

    ansible-playbook -i inv/txm-local txm.yml
N.B. This expects several development tools to be installed on the server for building the dependencies, and these aren't handled by this play outside of python2.7, python-dev, python-pip and libatlas3-base. You may need to install additional packages on the server if the play fails.

For those using Debian 8.7 we ship with pip packages and wheels that can be utilised for offline installs (where the host has no access to external pip repos):

    ansible-playbook -i inv/txm-local txm.yml -e "local=true"
For a number of packages we utilise python-dev to build them during the play, as the wheels don't function correctly, however python-dev can be removed from the server at the end of the play by using:

    ansible-playbook -i inv/txm-local txm.yml -e "local=true" -e "remove_python_dev=true"
Be careful with this!

We also ship with a Vagrantfile under vagrant/, which can be used for testing deployments against:

    ansible-playbook -i inv/txm-local txm.yml -u vagrant -e "local=true"
