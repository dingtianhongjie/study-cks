# マニュアル

https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/

# 学習ログ

serviceaccountを作成する。
```
$ kubectl create sa sa01
serviceaccount/sa01 created
$ kubectl get sa
NAME      SECRETS   AGE
default   1         5d23h
sa01      1         6s
```

作成したserviceaccountを使ってAPI Serverに認証するためにトークンを作成する。