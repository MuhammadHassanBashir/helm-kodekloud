# helm-kodekloud


Helm chart Anatony
------------------

commands:
---------

helm create <chart-name>  it will create a skeleton structure of a helm chart. 
ls <chart-name>


-   move all kubernetes deployment or services files in "templete folder" of skeleton structure of a helm chart. once we place files here, our helm chart should be techinically ready to go.. or ready to install helm package.. 
-   if you set static name in your deployment file. then every time when you are installing helm with new release name it will give you the error.. that same name chart is already installed by helm. so "it must have different names each time you install a helm chart." 

so how do we resolve this..

    we templating language, basically hum helm ko dye gye k name wo hi hoga jo user set kry ga...

    @deployment-file

    metadata:
        name: {{.Release.Name}}-(or anyother name)   like    name: {{.Release.Name}}-nginx

    "for making each release unique. you can use the same techique with all kubernetes objects.. like service... configmap, secret so on and so far.... 
     
     because helm does not allow to deploy 2 object with same name.... 

similar to name their are other object such as...

-   Release.Name
-   Release.Namespace
-   Release.IsUpgrade
-   Release.IsInstall
-   Release.Revision
-   Release.Service

or you can refer to any values from the chart.yaml file like...

    Chart.Name
    Chart.ApiVersion
    Chart.Version
    Chart.Type
    Chart.Keywords
    Chart.Home

or even details of kubernetes cluster.. like

    Capabilities.KubeVersion
    Capabilities.ApiVersion
    Capabilities.HelmVersion
    Capabilities.GitCommit
    Capabilities.GitTreeState
    Capabilities.GoVersion

or any thing starts with "values" or something property defined in the values.yaml file.

    like: 
    -   Values.replicaCount
    -   Values.image

    values are user define or 3 things are build in is lye in k 2 spell k first word capital ma ha..


hum deployment of service files ma kafi cheezo ko user define bna sakhty hn, jis sa user jo information dye ga or use horhi hogi file ma.

like:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Release.Name}}-nginx
  labels:
    app: nginx
spec:
  replicas: {{.Values.replicaCount}}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: {{.Values.image}}
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: {{.Release.Name}}-svc
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer

  ab ap deployment or service file ma jaha per b "values" use kry gye uski information ap helm chart install krty time "--set" use krty howy deployment or service file ma dye sakhty hn.. y as user input k toor per use horha hoga..

  like:

   - helm install <release-name> <chart-name>
        --set replicaCount=2
        --set image=nginx

    or 

    - we can define its values in "values.yaml" file...

ab sir khty hn k deployment file ma images ko "tag , latest" hn k b zaroor hoti hn or y sab values hum values.yaml file k ma dictionary ko use krty howy kesy dye sakthy hn...
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Release.Name}}-nginx
  labels:
    app: nginx
spec:
  replicas: {{.Values.replicaCount}}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: {{.Values.image.repositary}}:{{.values.image.tag}}  --> here values represent getting values from values.yaml file.. image represent values.yaml file image section... and repositary represent values.yaml file image repositary..  and tag represent values.yaml file image tag..
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: {{.Release.Name}}-svc
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer



values.yaml file..  is sa b hum deployment or service or other kubernetes objects ko values pass kr sakthy hn...
------------------

    replicaCount: 2
    image:
        repositary: nginx
        pullPolicy: IfNotPresent
        tag: "1.16.0"
    

    image: {{.Values.image.repositary}}:{{.values.image.tag}}  --> here values represent getting values from values.yaml file.. image represent values.yaml file image section... and repositary represent values.yaml file image repositary..  and tag represent values.yaml file image tag..

ab sir khty hn jb ap helm run krty hn tu wo 

    deployment file(release details) + values from values.yaml file + chart details leta ha or aik final complete deployment file bna kr usy deploy krta ha..

Making sure chart is working as intended
------------------------------------------

lets make sure the chart is working as intended. 

so there are 3 ways to verify your Helm charts before installing them..

-   "Linting" helps us verify that the chart and Yaml format is correct.

-   secondly verify the templates helps us make sure that the templating part is working as expected.

-   Third option is the dry run option, which helps us make sure that the chart works well with kubernetes itself.

or hum dekhy gye k hum kis tarha sa make sure kry gye k humra chart correctly built hoga without formating errors or wrong values etc.. mean hum helm deploy krna sa phily error ko file sa kis trha sa catch kry gye...

so y kam hum "Helm lint" command ko use krty howy ker sakhty hn..

    like:
        "helm lint <chart-name>"

- ab sir khty hn k agr hum chahty hn k hmara helm deploy na ho or hum deploy hony sa phily hi pta chal jye k helm deployment file ki values values.yaml or chart sa kya la rha ha. tu wo kam hum ker sakthy hn "helm template" k through..

command:    

    "helm template <chart-name>   . y apko ready to deployment file dekhy ga.. 

    The helm template command renders a chart template locally and displays the outputs.. file ma release name user define input name for deployment show krwa rha hoga...

"remember agr apki template files ma error hoga or ap helm template use kry gye tu wo apko error dekhy ga.. or obvious ni ha k wo exact error dekhy ..

y basically deploy krny sa phily apko output(final file) dekhy gi jis ko ap dekh kr indaza lga sakhty hn k values.yaml file or chart sa values yha kya arhi hn...

"agr ap na exact error dekhna chahty hn k jis file ma arha ha tu ap is command ko use kersakhty hn"  - 

    "helm template <chartname> --debug"   y apko exact file or uska error dekhy gi..


so we have verified that the YAML format is correct using the "helm lint" command and we have verified that helm templating is working as expected using the "helm template" command.. 

"these 2 steps help us catch a lot of errors". however there are still other error that may not be caught by these commands..

example ap na deployment.yaml file ma pod section ma under "spec" "containers"  use krny k bjye singular form "container" use ki ha tu. ap "helm link" or "helm template" k through h error find ni kr pao gye... because yaml or template format correct ha... or is error k hum kubernetes hi bta sakhta ha..

so use "--dry-run" with helm command. like "helm install <release name> <chart name> --dry-run"


Function in helm
-----------------------


we have seen that template and values.yaml file together create a valid manifest file.

"kya ho k values.yaml file ma feild hi set na ho.... like  

values.yaml file..  
------------------

    replicaCount: 2
    image:
        repositary: "empty"
        pullPolicy: IfNotPresent
        tag: "1.16.0"

        so isko use krty howy jo b deployment manifest file create hogi us ma b "image section empty hoga.." so jo pod crete hoga wo masla kry ga..


    like:


apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Release.Name}}-nginx
  labels:
    app: nginx
spec:
  replicas: {{.Values.replicaCount}}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: "empty"
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: {{.Release.Name}}-svc
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer


"so is masly k hal k lye humy kuch is tarha sa krna ha k agr "users user-define info" ni deta tu, values.yaml k through y manual command k through tu "default values ajye" mean fall back default values per hojye... tky pod chal sakhy.

"the logic would be if the user provided something in the values.yaml file then use that if there was not a value provided" and we can do such a thing using "functions"

what are functions
------------------

programming language ma 

    upper("helm") ==> "HELM"
    trim(" helm ") ==> "helm"

function input value ko leta ha isko process krta ha or phir output deta ha. functions in helm help transform data from one format to another..

There are just a few of the many string fucntions available in helm.

- abbrev
- abbrevboth
- camelcase
- cat
- contains
- hasPrefix
- hasSuffix
- indent
- initails
- kebabcase
- lower
- nindent
- nospace
- plural
- print
- printf
- printin
- quote
- randAlpha
- randAlphaNum
- randAsail
- randNUmeric
- repeat
- replace
- shuffle
- snakecase
- squote
- substr
- swapcase
- title
- trim
- trimall
- trimPrefix
- trimSuffix
- trunc
- untitle
- upper
- wrap
- wrapWith

these are string functions.. 

apart from string functions. there are other types of functions..

like:

- Cryptographic
- Date
- Dictionaries
- Encoding
- File Path
- kubernetes
- Chart
- Logic and flow
- Lists
- Math
- Network
- Reflection
- Regax
- URL
- String 
- Type conversion

so get back to our discussion k agr user values values.yaml sa or directly commmand k through ni dy rha tu deployment or pod ni chaly ga. so wo default values utha lye tky deployment or pod run ho.. 

like:
-----

ap deployment ma default values k lye asy mention ker sakhty hn...

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Release.Name}}-nginx
  labels:
    app: nginx
spec:
  replicas: {{.Values.replicaCount}}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: {{ default "nginx" .Values.image.repositary}}   --> mean agr user values.yaml file  ma values ni dye ga tu values deployment ko ni mily gi user ki so wo default set values use ker sakhta ha..
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: {{.Release.Name}}-svc
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer

Pipelines
-----------

echo "abcd" | tr a-z A-z

ABCD

echo will print abcd and pipe take the output and "tr" is for translate. so pipe take the output as input and tr will translate it.

so hum function ma dekha tha k hum kis tarha sa function ko pass kr sakhty hn... 

  "{{upper .Values.image.repository}}  --> image: NGINX..

  but we can do the same thing with other way.. one that you will see most commonly used is using through pipelines...

  like:

  "{{ .Values.image.repository | upper}}  --> image: NGINX.."

  both do the same thing....

  with pipelines, you can now pipe multiple functions one after the other.

    "{{ .Values.image.repository | upper | quote}}  --> image: "NGINX""

    or 

      "{{ .Values.image.repository | upper | quote | shuffle}}  --> image: GN"XNI""

Conditions
---------------------

values.yaml file...
------------------

replicaCount: 2
image: nginx

orgLabel: payroll


deploymentfile
--------------

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Release.Name}}-nginx
  {{ if.Values.orgLabel }}    --------------> here we apply conditional statement. k agr value.yaml file "orgLabel" exist krta ha. then next label values.yaml as label ki value get kry ga...
  labels:
    org: {{ .Value.orgLabel }}
  {{ end }}  
spec:
  replicas: {{.Values.replicaCount}}
  selector:
    matchLabels:
      app: nginx

final file y bnye gi..
-----------------------

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Release.Name}}-nginx
  
  labels:
    org: payroll   ==> values.yaml sa information ai ha..
  
spec:
  replicas: {{.Values.replicaCount}}
  selector:
    matchLabels:
      app: nginx


lets take more details in conditional statement.

ab sir khty hn final file ma condition statements ki jaga empty space ajye gi.. 

"so to resolve this, we can add a dash right after the curly braces to indicate Helm to trim those out when the files are generated.

like:
----

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Release.Name}}-nginx
  {{- if.Values.orgLabel }}    --------------> here we apply conditional statement. k agr value.yaml file "orgLabel" exist krta ha. then next label values.yaml as label ki value get kry ga...
  labels:
    org: {{ .Value.orgLabel }}
  {{- end }}  
spec:
  replicas: {{.Values.replicaCount}}
  selector:
    matchLabels:
      app: nginx

result:
------  you will get the result like this. extra spaces will trimmed it out...

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Release.Name}}-nginx 
  labels:
    org: payroll   ==> values.yaml sa information ai ha..  
spec:
  replicas: {{.Values.replicaCount}}
  selector:
    matchLabels:
      app: nginx


- ab sir khty hn k jis trha sa programming language ma, "if, else if , else" hota ha isi tarha helm ma b hota ha..

like:
-----

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Release.Name}}-nginx
  {{- if.Values.orgLabel }}    --------------> here we apply conditional statement. k agr value.yaml file "orgLabel" exist krta ha. then next label values.yaml as label ki value get kry ga...
  labels:
    org: {{ .Value.orgLabel }}
  {{- else if eq .Values.orgLabel "hr"}}
  labels:
    org: human resources
  {{- end }}  
spec:
  replicas: {{.Values.replicaCount}}
  selector:
    matchLabels:
      app: nginx

there are many other functions like this...

- eq equal
- ne not equal
- lt less than
- le less than or equal to 
- gt greator than or equal to
- not   negation
- empty value is empty

hum conditional statement ko different conditions k accordingly use kr sakhty hn...

like hum chahty hn k agr value.yaml ma service account k lye "create: true" set hoga tb hi service account create hoga..

values.yaml
-----------

serviceAccount:

  create: true

serviceaccount file:
--------------------
{{- if .Values.serviceAccount.create}}
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ .Release.Name}}-rebot-sa
 {{- else}} 


with Blocks
-----------
Setting scope using with
------------------------

lets now talk about scope...

let say we have a configmap. jis ko info values.yaml sa chahye.. 

values.yaml:
-----------

values.yaml ma information dictionary form ma ha..

app:
  ui:
    bg: red
    fg: black
  db:
    name: "users"
    conn: "mongodb://localhost:27020/mydb"

configmap:
---------

apiVersion: v1
kind:   configmap
metadata:
  name: {{ .Release.Name}}-appinfo
data:
  backgroup: {{ .Values.app.ui,bg}}
  foreground: {{.Values.app.ui.fg}}
  database: {{ .Values.app.db.name}}
  connection:   {{ .Values.app.db.conn}}  all will be getting the values from values.yaml file..

"so the dot as we discuss earliar is a reference to the current scope,  so everything falls under the root scope..

"dot mean under root scope..."

so dot because root scope ha. so isky under "release" or "values" arha ha..

or "release k under name" or "value k under app" or then app k under "ui" or "db", or "ui k under bg , fg" or "db k under name, conn" arhy hn...

is traha sa aik tree create horhi ha... "so that's kind of scope hierarchy"

"if you donot set the scope so the current scope by default set to "root".

that is why to access any of these objects, you must traverse all the way from the root scope..

- "ap na ab upper ".Value.app" ki multiple duplication dekhi hn. ap isko avoid ker sakhty hn... by setting a scope using a "with block"

like:
----

apiVersion: v1
kind:   configmap
metadata:
  name: {{ .Release.Name}}-appinfo
data:
  {{- with .Value.app}}         --------> current scope
    {{- with .ui}}
    backgroup: {{ .bg}}
    foreground: {{.fg}}
    {{- end}}
    {{- with .db}}
    database: {{ .name}}
    connection:   {{ .conn}}  all will be getting the values from values.yaml file..
    {{- end}}
  {{- end}}


  ab sir na bty ha k upper hum na dekha k root scope k under "release" or "values" arhy thy.. ab ap aik scope ma rakhty howy dosary scope ki values ko simply call kery gy tu error dye ga.. because by default her koi apny scope tk bound ha..

"  like values scope agr ap release ko simply call kry gye tu wo apko error dy ga. 

like this..
----------

apiVersion: v1
kind:   configmap
metadata:
  name: {{ .Release.Name}}-appinfo
data:
  {{- with .Value.app}}         --------> current scope
    {{- with .ui}}
    backgroup: {{ .bg}}
    foreground: {{.fg}}
    {{- end}}
    {{- with .db}}
    database: {{ .name}}
    connection:   {{ .conn}}  all will be getting the values from values.yaml file..
    {{- end}}
  release: {{. Release.Name}}   ---> it will give you error. becasue values.app k scope k under ha hi ni.. 
  {{- end}}

  it will give you error. becasue values.app k scope k under ha hi ni.. 
  
  "but kabi kabi aik scope ma dosary scope ki value ko lana parta ha ap wo  "root scope" k through la sakhty hn.. or root scope ki representation. "$" ha


so ap y kam asy ker sakhty hn...   "calling other scope value in a scope"
--------------------------------
apiVersion: v1
kind:   configmap
metadata:
  name: {{ .Release.Name}}-appinfo
data:
  {{- with .Value.app}}         --------> current scope
    {{- with .ui}}
    backgroup: {{ .bg}}
    foreground: {{.fg}}
    {{- end}}
    {{- with .db}}
    database: {{ .name}}
    connection:   {{ .conn}}  all will be getting the values from values.yaml file..
    {{- end}}
  release: {{ $.Release.Name}}   --> is tarha sa ap other scope ki value ko is scope ma get kr sakhty hn..
  {{- end}}

Loops and ranges
----------------

lets now talk about loops and ranges..

in a programming world, loops use to execute the code within a same block multiple times.. 

so we can use for loop in helm using the "range" operator..

let say k hum na config map create krna ha jo values.yaml file sa values ko la kr apny data section ma rakhy ga. ab values aik sa zayada ha.. tu ap helm k configmap na range ko use kr sakhty hn as loop. jo k one by one ker k values ko la rha hoga..

remember: programming ma iteration ki representation "i" sa horhi hoti ha... but helm ma representation "." sa horhi hoti ha...

values.yaml
-----------

regions:
  - ohio
  - newyork
  - ontario
  - london
  - singapore
  - mumbai

configmap
---------

apiVersion: v1
kind: ConfigMap
metadata:
  name: {{.Release.Name}}-regioninfo
data:
  regions:
  {{- range .Value.regions}}   ---> ab y value.yaml file ma region section sa region ko one by one ker k pick kry ga or configmap ma dye ga...
  - {{ . | quote}}              ----> . yha per iteration k lye use horha ha. or jesy hi koi value aye gi pipe usko as input lye ga or then "quote" function k through is per quotation lga dye ga.. and so on and soo farr.
  {{- end}}


Named template
--------------

reuse code and remove duplications. how can we make it simple by using named template.... so we can remove duplication line with name template...


we can move the repeated lines in "_helper.tpl" file. Now the underscore in the file name tell helm not to consider this file as a usual template file. 

because we know helm read all the file in "template" directoy and convert into kubernetes manifests. so all files starting with an underscore are skipped from being created or converted into a kubernetes manifest file. and you can also named this lines...

like
-----------

_helpers.tpl
------------

{{- define "labels" }}
  app.kubernetes.io/name: nginx
  app.kubernetes.io/instance: nginx
{{- end}}

so you can call this lines any where in the template..

like...
-------

---
apiVersion: v1
kind: Service
metadata:
  name: {{.Release.Name}}-svc
  labels:
    {{- template "labels"}}  ---> it will lines from "_helpers.tpl"
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer

final file would be
-------------------

apiVersion: v1
kind: Service
metadata:
  name: {{.Release.Name}}-svc
  labels:
      app.kubernetes.io/name: nginx
      app.kubernetes.io/instance: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer


_helpers.tpl
------------

{{- define "labels" }}
  app.kubernetes.io/name: nginx
  app.kubernetes.io/instance: nginx
{{- end}}

ab ap hard corded value ko user define b bna sakhty hn..

like this
-----------
_helpers.tpl
------------

{{- define "labels" }}
  app.kubernetes.io/name: {{ .Release.Name }}
  app.kubernetes.io/instance: {{ .Release.Name }}
{{- end}}

and to transfer the current scope to the template file... add a dot at the end of the template statement..

like :

apiVersion: v1
kind: Service
metadata:
  name: {{.Release.Name}}-svc
  labels:
    {{- template "labels" . }}  ---> it will lines from "_helpers.tpl"
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer



"ab sir khty hn k identation ko dekhna b buhat zaroori ha.. ap na jo repeated file "_helper.tpl" ma rhkhi hn in ko ap service or deployments file ma jaha per b call kro y same identation k sath hr jaha aye gi... but remember hr jaga aik jesi identation space ni hoti.. 

as krny sa apki files ni chaly gi.. so how we can resolve this issue.. we can fix this issue by using function name "indent"

like 
-----

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Release.Name}}-nginx
  labels:
    {{- template "labels" . }}
spec:
  replicas: {{.Values.replicaCount}}
  selector:
    matchLabels:
    {{- template "labels" . | indent 2}} ---> it will indent line of 2 space
  template:
    metadata:
      labels:
      {{- template "labels" . | indent 4 }}---> it will indent line of 4 space
    spec:
      containers:
      - name: nginx
        image: {{ default "nginx" .Values.image.repositary}}   --> mean agr user values.yaml file  ma values ni dye ga tu values deployment ko ni mily gi user ki so wo default set values use ker sakhta ha..
        ports:
        - containerPort: 80

"ab sir khty hn k y kam ni kry ga because template "function" ni ha "action" ha. so you need to use "include" instead of "template". because template is action and include is a function..


like 
-----

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{.Release.Name}}-nginx
  labels:
    {{- template "labels" . }}
spec:
  replicas: {{.Values.replicaCount}}
  selector:
    matchLabels:
    {{- include "labels" . | indent 2}} ---> it will indent line of 2 space
  template:
    metadata:
      labels:
      {{- include "labels" . | indent 4 }}---> it will indent line of 4 space
    spec:
      containers:
      - name: nginx
        image: {{ default "nginx" .Values.image.repositary}}   --> mean agr user values.yaml file  ma values ni dye ga tu values deployment ko ni mily gi user ki so wo default set values use ker sakhta ha..
        ports:
        - containerPort: 80

Chart Hooks
------------

ap jesy hi helm k through upgrage krty wo tu db automatically backup hoti ha phir upgrade hota ha.. tky incase of any issue hmry ps backup save ho...

so these extra actions are implemented with what is known as "hooks"..

"jb user helm insall or upgrade krta ha tu helm charts sa files or template ko "verify" kerta ha or phir "renders" manifest files ko unki final form ma lata ha.. or last ma services ko cluster ma install krta ha..

  helm upgrade ---> verify --> render --> upgrade..    

let say hmry requirement hn k chart upgrade hony sa phily backup lya jye... tu isky lye wo "pre-upgrade hook" ko use krty hn.. so for our case pre upgrade hook phily backup lye ga phir usky bd upgrade kry ga...

        helm upgrade ---> verify --> render --> pre-upgrade hook --->upgrade..    

so after the upgrade hum chahty hn k some kind of cleanup activity ho or kisi ko email send krni ha... usky lye hum "post upgrade hook" use kry gye..

        helm upgrade ---> verify --> render --> pre-upgrade hook --->upgrade --> post upgrade hook..  

"along with we have other hooks. you can use this hooks during "installation", "deletion", "pre rollback".. 

like:
----

helm upgrade ---> verify --> render --> pre-install hook --->upgrade --> post install hook.. 
helm upgrade ---> verify --> render --> pre delete hook --->upgrade --> post delete hook.. 
helm upgrade ---> verify --> render --> pre roleback hook --->upgrade --> post rollback hook.. 

let see how to configure hooks:
--------------------------------

hb sir khty hn k let say ap na upgrade sa phily backup lena ha y kam through pre upgrade hook sa ap krna chahty ho.. so apky ps backup k lye backup.sh file ha ap isko kubernetes ma kesy run kro gye... isko ap pod ma run kr sakhty hn.. but sir khty hn pod tu chalta rhy ga.. jb k hum na tu sirf isko aik br run krna ha... 

so kubernetes ma iskly "job" name k object hota ha. ap isko pre-upgrade hook ma use kr sakhty hn.. mean job backup k lye run hogi jb job complete hojy gi or backup bn jye ga tb helm upgrade hoga. 

ab sir khty hn k apki sari file. "template" name k folder ma hoti hn. ap helm ko kesy btao gye k template folder ma kon si file manifest k lye hn or kon si pre-upgrade hook ma use honi ha...

is solution k lye hum pre-upgrade hook ki file ma(mean job file) ma "annotation" use kerty hn..

like:
----

apiVersion: batch/v1
kind: job
metadata:
  name: {{ .Release.Name}}-nginx
  annotations:
    "helm.sh/hook": pre-upgrade    --> this annotation will define k y pre upgrade hook ma job use horhi hogi..
spec:
  template:
    metadata:
      name: {{. Release.Name}}-nginx
    spec:
      restartPolicy: Never
      containers:
      - name: pre-upgrade-backup-job
        image: "alpine"
        command: ["/bin/backup.sh"]


ab ap na multiple job ko exist kr rhy hn same preupgrade hook ma or ap is cheez ko kesy make sure kry gye k kon si job phily execute ho. y kam ap "weight" k through ker sakhty hn.. apko jobs ki manifest file ma annotation k sath weight b btana ha.. is sa y hoga ka helm jo pta chal jye ga k kis job ko phily run krna ha.. 


like:
-----

apiVersion: batch/v1
kind: job
metadata:
  name: {{ .Release.Name}}-nginx
  annotations:
    "helm.sh/hook": pre-upgrade    --> this annotation will define k y pre upgrade hook ma job use horhi hogi..
     "helm.sh/hook-weight": "5"       mean helm is job ko 5th no per run kry ga..   
spec:
  template:
    metadata:
      name: {{. Release.Name}}-nginx
    spec:
      restartPolicy: Never
      containers:
      - name: pre-upgrade-backup-job
        image: "alpine"
        command: ["/bin/backup.sh"]


more usefull hooks:
------------------

"helm.sh/hook-delete-policy": hook-succeeded
"hook-failed"
"before-hook-creation"

packaging and uploading charts
------------------------------

we know have our chart build and ready to be shared with the world.

- 1st we package it.
- and then we upload it to the online chart repositary..

"let say ap na "nginx-chart" name ka chart create kya ha... ab apna na isko package krna ha..

run this command..

  helm package ./nginx-chart

- this package the chart into "archive format" .tgz and name n

0 "now we have package the tassk so next step isko upload it to chart repoitory..

one of the ways to make download safer isto graphically sign the files and package...

chart devopler chart ko sign krta ha.. or jb iako koi or user download kr ha tu isko chart per sign milta ha ji sa y idea hota ha k y chart sign h chart developer k traf sa...


first need a private key:

-------------------------

gpg --quick-generate-key "john smith"

this could uploaded to an open PGP key server like "keyserver.ubunutu.com"

- command ko make more key..
============================

gpg --full generate-key "john smith"

gpg --export-secret-keys >1/.gnupg/secring.gpg

now we have key ready and package it with using helm package command but with the sign option

  helm package --sign --key 'john Smith' --keyring~/.gnupg/secring.gpg ./nginx-chart

  gpg --list-keys


uploading chart to some online repositary..
-------------------------------------------

you should copy both the tgz chart archive and also the tgz.proc .provenance file.

jb user k ps y files ko gi tu wo "helm verify <chart-name>" sa iski verification kr sakhty hn..

"helm verify --keyring ./mypublickey <chartname>"
"gpg --recv-key --keyserver keyserver.ubuntu.com <>"

uploading chart
----------------

uploading chart so users can easily install through simple helm command...

now generally our chart repositories will contain these things..

- package chart --> .tgz file
- index.yaml file ---> the index file that container information about the chart repository, the chart it contains, checksums for our TGZ files, descriptions, and so on. "this is the file that helm will read when we add a new repository with a command like helm repo where we add a repo and point it to the chart repoistary"
- provenance file  ---> is case users want to verify cryptographic signature... so they make it sure that content is not uploaded by malicious attacker.


generate index.yaml file..
---------------------------

let say we have package the file with helm package command..

so list directory and move .tgz file in directory..

ls
mkdir nginx-chart-files
cp nginx-chart-0.1.0.tgz nginx-chart-0.1.0.tgz.provn  nginx-chart-files

now,

helm repo index nginx-chart-files/ --url https://example.com/charts

we know see the index.yaml file..

index.yaml file bta rhi hogi k chart ko kha sa download krna ha..

"now uplaod it to a chart repositary" ap isko gcp bucket ma , aws S3, digitalOcean ma update kr sakhty hn..

once chart upload. all that you need to share the url to those who download the chart

like:

helm repo add ..
helm repo list
helm install <release name> <chart name>
