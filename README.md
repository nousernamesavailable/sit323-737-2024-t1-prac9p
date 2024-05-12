# sit323-737-2024-t1-prac9p

Instructions for use: 
kubectl apply -f createStorageClass.yaml
kubectl apply -f createConfigMap.yaml
kubectl apply -f createMongoDbSecret.yaml 
kubectl apply -f createStatefulSet.yaml

kubectl get pods 
# (after a bit it'll show up) 

kubectl apply -f createHeadlessService.yaml

# we want one writer/primary and two readers
# so we set up mongo-0 as the primary, and 1 and 2 as the secondary readers. 

# open mongo-0 container, then mongo: 
kubectl exec -it mongo-0 bash
mongo

# paste the following:
rs.initiate(
   {
      _id: "rs0",
      members: [
         { _id: 0, host : "mongo-0.mongo.default.svc.cluster.local:27017" },
         { _id: 1, host : "mongo-1.mongo.default.svc.cluster.local:27017" },
         { _id: 2, host : "mongo-2.mongo.default.svc.cluster.local:27017" }
      ]
   }
)

# for mongo-1: 
kubectl exec -it mongo-1 bash
mongo
rs.slaveOk()

# and mongo-2:
kubectl exec -it mongo-2 bash
mongo
rs.slaveOk()

# back to mongo-0 - will add an entry:
use test
db.createCollection("tutorialspoint")
db.tutorialspoint.insert({"name" : "hi_is_maddie"})

# and test in mongo-1 and mongo-2, we check we can retrieve: 
use test
db.tutorialspoint.find()