---

copyright:
  years: 2018, 2019
lastupdated: "2019-06-03"

keywords: IBM Cloud Container Registry, Data Shield environment, datashield-barbican image, container image, Barbican, Registry, data in use, memory encryption, Intel SGX, Fortanix,

subcollection: RegistryImages

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:deprecated: .deprecated}
{:table: .aria-labeledby="caption"}

# Einführung zum Image `datashield-barbican`
{: #datashield-barbican_starter}

Dieses Container-Image bietet den verwendeten Daten durch die Barbican-Ausführung in der {{site.data.keyword.datashield_short}}-Umgebung Schutz. Weitere Informationen zu dem Service und die Bedeutung des Schutzes von "verwendeten Daten" finden Sie in der [{{site.data.keyword.datashield_short}}-Dokumentation](/docs/services/data-shield?topic=data-shield-about#about).
{: shortdesc}

Sie können auf die von {{site.data.keyword.IBM}} bereitgestellten Images über die Befehlszeile zugreifen. Informationen hierzu finden Sie in [öffentliche IBM Images](/docs/services/Registry?topic=registry-public_images#public_images).
{: tip}

## Image bereitstellen
{: #datashield-barbican-deploy}

Mithilfe der folgenden Kubernetes-Podspezifikationen können Sie das Image bereitstellen:

```
apiVersion: v1
kind: Pod
metadata:
  name: data-shield-barbican
  labels:
    app: data-shield-barbican
spec:
  containers:
  - name: data-shield-barbican
    image: <IMAGE_NAME>
    volumeMounts:
    - mountPath: /dev/isgx
      name: isgx
    - mountPath: /dev/gsgx
      name: gsgx
    - mountPath: /var/run/aesmd/aesm.socket
      name: aesm-socket
    env:
    - name: NODE_IP
      valueFrom:
        fieldRef:
          fieldPath: status.hostIP
    - name: NODE_AGENT_BASE_URL
      value: http://$(NODE_IP):9092/v1
    ports:
    - containerPort: 9311
      name: barbican
      protocol: TCP
  volumes:
  - name: isgx
    hostPath:
      path: /dev/isgx
  - name: gsgx
    hostPath:
      path: /dev/gsgx
  - name: aesm-socket
    hostPath:
      path: /var/run/aesmd/aesm.socket
---
apiVersion: v1
kind: Service
metadata:
  name: data-shield-barbican
  labels:
    app: data-shield-barbican
spec:
  type: LoadBalancer
  selector:
    app: data-shield-barbican
  ports:
    - name: barbican
      port: 9311
      targetPort: 9311
```
{: codeblock}

<table>
<caption>Tabelle 1. Erforderliche Eingabevariablen</caption>
  <tr>
    <th>Variable</th>
    <th>Beschreibung</th>
  </tr>
  <tr>
    <td><code>IMAGE_NAME</code></td>
    <td>Der Name des Image, das Sie bereitstellen möchten.</td>
  </tr>
  <tr>
    <td><code>NODE_IP</code></td>
    <td>Die Knoten-IP-Adresse Ihrer Barbican-Instanz.</td>
  </tr>
</table>

Das Bereitstellen und Starten des Containers kann einige Minuten in Anspruch nehmen. Die Barbican-Demonstrationsinstanz ist ohne Keystone-Authentifizierung konfiguriert. Anforderungen müssen den Header `X-Project-Id: 12345` enthalten.
{: tip}

## Geheimen Schlüssel erstellen
{: #datashield-barbican-secret}

Sie können einen geheimen Schlüssel in der Barbican-Demonstrationsinstanz erstellen, indem Sie den folgenden Befehl ausführen:

```
curl -sS -X POST -H 'content-type:application/json' -H 'X-Project-Id: 12345' -d \
        '{"payload": "my-secret-here", "payload_content_type": "text/plain"}' \
        http://<node>:9311/v1/secrets
```
{: codeblock}

Die Antwort enthält die URL für den erstellten geheimen Schlüssel. Sie können den geheimen Schlüssel abrufen, indem Sie den folgenden Befehl ausführen:

```
curl -sS -H 'X-Project-Id: 12345' <url>
```
{: codeblock}