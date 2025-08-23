Provision my home lab with K3s using the [k3s-ansible](https://github.com/k3s-io/k3s-ansible) collection

1. Install k3s-ansible collection `ansible-galaxy collection install git+https://github.com/k3s-io/k3s-ansible.git`
1. Create [inventory.yaml](https://github.com/k3s-io/k3s-ansible/blob/master/inventory-sample.yml) with [ansible_user](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#term-ansible_user) and [ansible_host](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#term-ansible_host)
1. Run --check `ansible-playbook -i inventory.yaml cluster.yaml --check --become`
1. Run `ansible-playbook -i inventory.yaml cluster.yaml --become`
