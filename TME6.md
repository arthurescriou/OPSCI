# TME6 : Scale your application

This TME aims to use and understand de scaling tools in kubernetes.

#### Exercise 1

Use your redis project from TME4.

Limit the available resources for the pods (add this to your deployment file) :

```yaml
resources:
  limits:
    cpu: 500m
  requests:
    cpu: 200m
```

Set the `replicas` for your deployments to 2.

Try to call your servers with :

```sh
curl ${adress_of_your_server}
```

Notice that the IDS must change : `It is working, good job ${UUID} ${DATE}`.

#### Exercise 2

Now we want to create a dynamic scaling behavior for our pods.

Set the replicas for your deployments to 2 to 10 using `minReplicas` and
`maxReplicas` instead of `replicas`.

And set `targetCPUUtilizationPercentage` to `50` to force the pod to scale when it is used.

We need now to stimulate the pod to force kubernetes to scale the pods up.

Create a script that call the service.

```sh

while true
do
  curl ${adress_of_your_server} &
  sleep 0.1
done
```

You can also change the rest call to heavier call to the server like database interaction (create record or read from the database).

In an other terminal observe the reaction of the cluster.

```sh
watch kubectl get deployments
```

If the number of pods does not change try to change some of the values previously set :

- `minReplicas`
- `targetCPUUtilizationPercentage`
- cpu limitaion to the pods
