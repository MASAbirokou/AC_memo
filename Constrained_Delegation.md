# Constrained Delegation abuse

![delegation](https://user-images.githubusercontent.com/85237728/158114156-6e934692-fdf4-4e26-b0b8-93f9e4da7e51.png)

> The constrained delegation privilege allows a principal to authenticate as any user to specific services on the target computer.
> That is, a node with this privilege can impersonate any domain principal (including Domain Admins) to the specific service on the target host.

上の画像で、SVC_INTがドメインコントローラに対してconstrained delegation privilegeを持っている。で、SVC_INTはadminになりすます（impersonate）ことが出来る。
