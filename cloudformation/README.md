# EC2 + S3 + CloudWatch Monitoring Stack

Template de CloudFormation que despliega una arquitectura de referencia en AWS,
aplicando buenas prácticas de seguridad y observabilidad.

## Qué incluye

- **EC2**: instancia con AMI resuelta automáticamente vía SSM Parameter Store
  (siempre la última Amazon Linux 2023 disponible, sin hardcodear IDs de AMI).
- **IAM Role de mínimo privilegio**: la instancia solo puede leer/escribir en
  su bucket S3 asociado y publicar métricas/logs — nada más.
- **S3**: bucket con versionado, cifrado por defecto (SSE-S3) y bloqueo total
  de acceso público.
- **CloudWatch Alarms**: alertas de CPU alta (>80% sostenido) y de status
  check fallido, notificando por email vía SNS.
- **Security Group**: acceso SSH restringido a un solo puerto (pensado para
  ajustarse a un CIDR específico en producción).

## Uso

```bash
aws cloudformation deploy \
  --template-file ec2-s3-monitoring-stack.yaml \
  --stack-name dev-cloudops-stack \
  --parameter-overrides \
      EnvironmentName=dev \
      KeyPairName=mi-keypair \
      AlarmNotificationEmail=tu-email@ejemplo.com \
  --capabilities CAPABILITY_NAMED_IAM
```

## Parámetros

| Parámetro | Descripción | Default |
|---|---|---|
| `EnvironmentName` | Prefijo de entorno (dev/staging/prod) | `dev` |
| `InstanceType` | Tipo de instancia EC2 | `t3.micro` |
| `KeyPairName` | Key pair existente para SSH | — |
| `AlarmNotificationEmail` | Email para alertas de CloudWatch | — |

## Por qué este diseño

Este template refleja el enfoque que aplico en entornos productivos: roles
de IAM acotados al mínimo necesario, cifrado y bloqueo de acceso público por
defecto en S3, y monitoreo activo desde el día uno — en vez de agregarlo
después como una idea tardía.
