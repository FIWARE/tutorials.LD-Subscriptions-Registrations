[![FIWARE Banner](https://fiware.github.io/tutorials.LD-Subscriptions-Registrations/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Relationships-Linked-Data.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.04.02_60/gs_cim009v010402p.pdf)

This tutorial discusses the usage of subscriptions and registrations within NGSI-LD and highlights the similarities and
differences between the equivalent NGSI-v2 and NGSI-LD operations. The tutorial is an analogue of the original
context-provider and subscriptions tutorials but uses API calls from the **NGSI-LD** interface throughout.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as
[Postman documentation](https://www.postman.com/downloads/).

# Start-Up

## NGSI-LD Smart Supermarket

**NGSI-LD** offers JSON-LD based interoperability used for Federations and Data Spaces. To run this **NGSI-LD** tutorial for **NGSI-v2** developers, use the `NGSI-v2` branch.

```console
git clone https://github.com/FIWARE/tutorials.LD-Subscriptions-Registrations.git
cd tutorials.LD-Subscriptions-Registrations
git checkout NGSI-v2

./services create
./services start
```

| [![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.08.01_60/gs_cim009v010801p.pdf) | :books: [Documentation](https://github.com/FIWARE/tutorials.LD-Subscriptions-Registrations/tree/NGSI-v2) | <img src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/postman.svg" height="15" width="15"> [Postman Collection](https://fiware.github.io/tutorials.LD-Subscriptions-Registrations/) | ![](https://img.shields.io/github/last-commit/fiware/tutorials.LD-Subscriptions-Registrations/NGSI-v2)
| --- | --- | --- | ---

---

## License

[MIT](LICENSE) © 2020-2024 FIWARE Foundation e.V.
