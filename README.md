# Steps To Build Remote Execution Node

- On one of the controller VMs:
- Install the community.crypto collection

    `ansible-galaxy collection install community.crypto`

- Pull the repo:

    [https://github.com/derekwaters/ansible_receptor_build](https://github.com/derekwaters/ansible_receptor_build)

- Override or modify the vars in the playbook:

    `generate_receptor_install_bundle.yaml`

- Run the playbook:

       ansible-playbook -i localhost \
            -e controller_node_hostname=aap.localdomain \
            -e execution_node_hostname=securexec.localdomain \
            -e input_folder=/home/awx/ \
            -e output_folder=/var/tmp/receptor_install_bundle \
            -e signing_temp_path=/var/tmp/ \
            generate_receptor_install_bundle.yml


    In the output folder, a series of certs and yaml files will be created. Review them for correctness, then zip up the contents and transfer (using whatever secure mechanism is available) to your execution node.
- On the execution node:
- Unzip the bundle
- Install the requirements:

    `ansible-galaxy collection install -r requirements.yml`

- Execute the install playbook with the provided inventory

    `ansible-playbook -i inventory.yml -K install_receptor.yml`

- Ensure the install runs correctly.
- Check the receptor status with:

    `sudo systemctl status receptor`

- Back on the control node, you can validate that the receptor connection has been established by running:

    `podman exec receptor receptorctl status`

  Note that in later versions of AAP (2.6-3) receptorctl is no longer bundled in the receptor container. In which case you can run it from a controller node:

    `podman exec automation-controller-web /var/lib/awx/venv/awx/bin/receptorctl --socket /run/receptor/receptor.sock status`

- And ensuring that the new node is displayed.

- Finally, on the controller node, create the execution instance by running:

    `podman exec automation-controller-task awx-manage provision_instance --hostname <remotehostname> --node_type execution`

- After a period of a few minutes to run health checks, check the instance status in the AAP console.
