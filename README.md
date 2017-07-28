# ohze test on Rancher or Docker Swarm

1. install docker
Optional: install [jq](https://stedolan.github.io/jq/)
(run `brew install jq` if you use MacOS)
2. install [Rancher]
```
sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server:stable
```
3. download [rancher CLI](https://github.com/rancher/cli/releases)
4. [config](http://rancher.com/docs/rancher/v1.6/en/cli/) rancher cli

5. Prepare Rancher
In Rancher UI:
+ add a Cattle environment `Default`
+ add 2 Host.
One with label `ambry.id=1`, `ambry.bootstrap.upgrade=1`
The other with label `ambry.id=2`

6. run the following rancher CLI commands:
```bash
# add `zk` stack
rancher --wait stack create zk --start -f zk/docker-compose.yml
# rancher inspect `rancher inspect zk |jq .serviceIds[0] -r` | jq .state
rancher --wait stack create ambry-mock --start -f ambry-bootstrap-upgrade/ambry-mock/docker-compose.yml
rancher --wait stack create ambry-bootstrap-upgrade --start -f ambry-bootstrap-upgrade/docker-compose.yml
rancher --wait stack create helix --start -f helix/docker-compose.yml
rancher --wait stack create ambry --start -f ambry/docker-compose.yml
rancher rm ambry-mock
rancher --wait stack create ambry-frontend --start -f ambry-frontend/docker-compose.yml
```

Bonus:
create a service in rancher with image `sandinh/helix` then `docker exec -it <id> sh` to it to run helix commands, ex:
```bash
helix-admin.sh --zkSvr zoo1.zk:2181,zoo2.zk:2181,zoo3.zk:2181 --dropCluster Ambry_Ohze
```
