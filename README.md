# Certified Kubernetes Administrator (CKA) - V1.29

The repository will help you for taking the Certified Kubernetes Administrator (CKA) exam using online resources, especially using resources from [Kubernetes Official Documentation](https://kubernetes.io).

The references were selected for the [Exam Curriculum 1.29](https://github.com/cncf/curriculum).

Please, feel free to contribute whether something is not up-to-date, should be added or contains wrong information/reference.

## Exam Details

The exam is kind of "put your hands on", where you have some problems to fix within 120 minutes.

The exam focuses on 5 key domains:

- 25% on Cluster Architecture, Installation & Configuration
- 15% on Workloads & Scheduling
- 20% on Services & Networking
- 10% on Storage
- 30% on Troubleshooting

The exam consists of 15-20 performance-based tasks that you need to complete within 2 hours.

## Tips for the Exam

1. This is a hands-on exam. So make sure you're very comfortable with the `kubectl` command and practice typing commands quickly and accurately. Use aliases if needed.

    ```bash
    alias k=kubectl
    ```

2. Leverage imperative commands: Refer to the kubectl cheatsheet for quick assistance.

3. Set auto-completions and kubectl alias: At the beginning of the exam, configure auto-completions and aliases to save time.

4. Always remeber to Switch cluster context and delete resources quickly using the --force flag wherever it goes wrong

5. Rather than writing YAML from scratch, use --dry-run with the createand runcommands to generate YAML. Edit it to your needs using vimbefore running e.g.

    ```bash
    export do="-o yaml --dry-run=client"
    export now="--grace-period 0 --force"
    ```

The course also provides a browser terminal. Practicing above question set will give you an idea of CKA.

## For Additional Practice Use below platform

- [Killer.sh - CKA Simulator](https://killer.sh/cka)
- [Kubernetes the Hard Way by Kelsey Hightower](https://github.com/kelseyhightower/kubernetes-the-hard-way)
