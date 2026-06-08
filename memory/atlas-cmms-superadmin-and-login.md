---
name: atlas-cmms-superadmin-and-login
description: Atlas CMMS login/auth quirks - super admin account, web login type, password reset requires SMTP
metadata:
  type: reference
---

Atlas CMMS auth no `/auth/signin`: valida email+senha E exige que o `type` enviado bata com a role do usuário (`UserService.signin`, lança 403 se não bater). A tela web SEMPRE envia `type: 'client'` (`JWTAuthContext.tsx`), então só usuários com role `ROLE_CLIENT` logam pela UI. Contas com `role_type=0` (ex.: `superadmin@test.com` seed `pls_change_me`, e `otavio@venttos.com.br`) são tipo SUPER_ADMIN e NÃO logam pela tela web — só via API com `type: "SUPER_ADMIN"`.

Reset de senha (`/auth/resetpwd`) retorna 406 se `ENABLE_EMAIL_NOTIFICATIONS=false` (precisa SMTP configurado) e falha se a conta não existir. SMTP via env: `SMTP_USER/SMTP_PWD/SMTP_FROM/SMTP_HOST/SMTP_PORT` no `.env` do servidor (Gmail porta 587 STARTTLS). O nome de exibição do remetente vem do brand config, não de env var. Template `reset-password.html` já existe.

Tabela de usuários: `own_user` (senha em BCrypt, coluna `password`; login por `email`, salvo lowercase). Ver [[atlas-cmms-selfhosted-deploy]].
