# Kubernetes 
## Logging options

When you run [`kubectl logs`](https://kubernetes.io/docs/user-guide/kubectl/v1.9/#logs) as in the basic logging example, the kubelet on the node handles the request and reads directly from the log file, returning the contents in the response. **Note:** currently, if some external system has performed the rotation, only the contents of the latest log file will be available through `kubectl logs`. E.g. if there’s a 10MB file, `logrotate` performs the rotation and there are two files, one 10MB in size and one empty, `kubectl logs` will return an empty response. An example of examining logs follows:

```
kubectl logs <podname> -c <containername> (eg: kubectl logs person-api-1530694245-z742l -c person-api)
```

There is also a tool, kubetail, which can tail the logs for a Kubernetes cluster. This can be parsed into a log file locally or ported externally. 

Kubetail can be found here:
- [https://github.com/johanhaleby/kubetail](https://github.com/johanhaleby/kubetail)

Information on basic logging in Kubernetes can be found here:
- [https://kubernetes.io/docs/concepts/cluster-administration/logging/](https://kubernetes.io/docs/concepts/cluster-administration/logging/)

Some notes on logging to Elasticsearch are here:
- [http://blog.wercker.com/kubernetes-logging-tutorial](http://blog.wercker.com/kubernetes-logging-tutorial)