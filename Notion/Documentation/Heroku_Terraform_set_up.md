# Heroku/Terraform set up

[https://devcenter.heroku.com/articles/using-terraform-with-heroku](https://devcenter.heroku.com/articles/using-terraform-with-heroku)

## Set Up de Terraform :

- Télécharger Terraform et l'installer
    - Mettre en place un Back-end en remote avec aws S3 afin de pouvoir sauver l'infrastructure Terraform. Pour cela, il faut spécifier dans le fichier de configuration (main.tf) l'information suivante :

    ```
    terraform {
      backend "s3" {
        bucket = "mybucket"
        key    = "path/to/my/key"
        region = "us-east-1"
      }
    }
    ```

    ((Vous pourrez choisir un plan gratuit ou bien un plan premium payant

    Nous opterons pour le plan standard 0 à 50$/mois pour bénéficier d'un Rollback possible de 4 jours))

    ### Autorisation :

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::mybucket"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::mybucket/path/to/my/key"
    }
  ]
}
```

## Approvisionnement d'une application vide

Ajoutez une ressource d'application Heroku à votre fichier [main.tf](http://main.tf) : 

```
variable "app_name" {
  description = "Name of the Heroku app"
}

resource "heroku_app" "example" {
  name   = "${var.app_name}"
  region = "us"
}
```

Cela indique à Terraform de provisionner une app Heroku vide avec un nom que vous fournirez à la commande terraform apply.
Indépendamment du nom Heroku de l'application, l'identifiant de Terraform pour cette ressource particulière est heroku_app.example, le type de ressource et son nom. Cet identifiant peut être utilisé comme variable dans la configuration et pour des opérations telles que l'importation ou la visualisation de l'état.

### Planifier et appliquer :

Utilisez la commande terraform apply pour appliquer la configuration que vous avez sauvegardée dans [main.tf](http://main.tf/) :

```
$ terraform apply -var app_name=FruitNow
```

Une fois l'application terminée avec succès, les ressources créées par Terraform seront présentes dans le compte Heroku associé à la clé d'autorisation de Terraform.
Consultez l'état actuel de Terraform pour voir ce qui a été créé :

```
$ terraform show
```

## Déploiement de code dans une application

Pour utiliser Terraform afin de déployer du code dans l'application vide que vous avez provisionnée dans Provisioning an empty app, mettez à jour [main.tf](http://main.tf/) pour qu'il corresponde à ce qui suit :

```
provider "heroku" {
  version = "~> 2.0"
}

variable "app_name" {
  description = "Name of the Heroku app"
}

resource "heroku_app" "example" {
  name = "${var.app_name}"
  region = "us"
}

# Build code & release to the app
resource "heroku_build" "example" {
  app = "${heroku_app.example.name}"
  buildpacks = ["https://github.com/mars/create-react-app-buildpack.git"]

  source = {
    url = "https://github.com/mars/cra-example-app/archive/v2.1.1.tar.gz"
    version = "2.1.1"
  }
}

# Launch the app's web process by scaling-up
resource "heroku_formation" "example" {
  app        = "${heroku_app.example.name}"
  type       = "web"
  quantity   = 1
  size       = "Standard-1x"
  depends_on = ["heroku_build.example"]
}

output "example_app_url" {
  value = "https://${heroku_app.example.name}.herokuapp.com"
}
```

Appliquez la configuration, en passant encore une fois le nom de votre application comme variable d'entrée :

```
$ terraform apply -var app_name=FruitNow
```

Après que terraform apply se soit terminé avec succès, visitez l'URL de votre application Heroku qui est disponible en tant que sortie de la configuration.

```
$ terraform output app_url
```

Si vous ne souhaitez pas conserver l'échantillon, vous pouvez utiliser Terraform pour tout nettoyer :

```
$ terraform destroy -var app_name=FruitNow
```

## Créer une application et un add-on

Ce bout de code crée une ressource App et une ressource Add-on dans une équipe Heroku et une région Heroku particulières :

```
variable "heroku_team" {
  description = "Name of the Team (must already exist)"
}

resource "heroku_app" "example" {
  name   = "${var.heroku_team}-example"
  region = "us"

  organization = {
    name = "${var.heroku_team}"
  }
}

resource "heroku_addon" "papertrail_example" {
  app  = "${heroku_app.example.name}"
  plan = "papertrail:choklad"
}
```

## Scaling de l'app

Ce bout de code met à l'échelle une ressource de formation pour mettre à l'échelle une application existante. Il exécute également une commande locale (un contrôle de santé du provisionneur) pour attendre que l'application soit lancée avec succès.

```
resource "heroku_formation" "example" {
  app        = "${heroku_app.example.name}"
  type       = "web"
  quantity   = 2
  size       = "Standard-1x"
  depends_on = ["heroku_app_release.example"]

  provisioner "local-exec" {
    command = "./bin/health-check http://${heroku_app.example.name}.herokuapp.com""
  }
}
```

### Créer un espace privé :

Ce bout de code crée une ressource Private Space, en s'assurant qu'elle se trouve dans la même région que les ressources AWS :

```
variable "heroku_enterprise_team" {
  description = "Name of the Enterprise Team (must already exist)"
}

variable "heroku_private_space" {
  description = "Name of the Private Space"
}

variable "aws_region" {
  description = "Amazon Web Services region"
  default = "us-east-1"
}

variable "aws_to_heroku_private_region" {
  default = {
    "eu-west-1"      = "dublin"
    "eu-central-1"   = "frankfurt"
    "us-west-2"      = "oregon"
    "ap-southeast-2" = "sydney"
    "ap-northeast-1" = "tokyo"
    "us-east-1"      = "virginia"
  }
}

resource "heroku_space" "example" {
  name         = "${var.heroku_private_space}"
  organization = "${var.heroku_enterprise_team}"
  region       = "${lookup(var.aws_to_heroku_private_region, var.aws_region)}"
}
```