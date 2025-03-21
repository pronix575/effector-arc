# Правила создания сервисов
1. новый сервис добавляется в папку services
2. если он связан с некой центральной сущностью, то добавляем в папку этой сущности

- nodesService
  - displayNodeService
  - displayNodesListService
  - editNodeService
    - editNodeService.models.ts
    - editNodeService.conainer.ts
    - editNodeService.types.ts
    - editNodeService.container.ts
    - editNodeService.api.ts
    - views 
      - EditNodeForm
        - EditNodeForm.tsx
        - EditNodeForm.styled.ts
        - EditNodeForm.types.ts
      - EditNodeModal.tsx
      - EditNodeModal.types.ts
      - EditNodeModal.styled.ts

### Что такое container
Контейнер связывает локальную модель сервиса с ее view.
Таким образом внутри контейнера может быть использована только локальная модель сервиса.
Если нужны данные с другой модели, то реэкспортим внутри локальной модели.

### deleteIndividualDeviceService.models.ts
Объявляются базовые компоненты сервиса, описывются связи внутри сервиса, экспортится объект модели с полями inputs и outputs 
```ts
import { createDomain, sample } from 'effector';
import { IndividualDeviceListItemResponse } from 'api/types';
import { deleteDevice } from './deleteIndividualDeviceService.api';

const domain = createDomain(
  'deleteIndividualDeviceService'
);

const $currentIndividualDevice = domain.createStore<IndividualDeviceListItemResponse | null>(
  null
);

const $isModalOpen = $currentIndividualDevice.map(Boolean);

const deleteDeviceModalOpened = domain.createEvent<IndividualDeviceListItemResponse>();
const deleteDeviceModalClosed = domain.createEvent();

const acceptDeleteDevice = domain.createEvent();

const deleteIndividualDeviceFx = domain.createEffect<
  number,
  void
>(deleteDevice);

const deletingComplete = deleteIndividualDeviceFx.doneData;

$currentIndividualDevice
  .on(deleteDeviceModalOpened, (_, device) => device)
  .reset(deleteDeviceModalClosed, deleteIndividualDeviceFx.doneData);

sample({
  source: $currentIndividualDevice.map((device) => device?.id),
  clock: acceptDeleteDevice,
  filter: (id): id is number => typeof id === 'number',
  target: deleteIndividualDeviceFx,
});

export const deleteIndividualDeviceService = {
  inputs: {
    deleteDeviceModalOpened,
    deleteDeviceModalClosed,
    acceptDeleteDevice,
    deletingComplete,
  },
  outputs: {
    $isModalOpen,
    $currentIndividualDevice,
    $loading: deleteIndividualDeviceFx.pending,
  },
};
```

### deleteIndividualDeviceService.api.ts
Описываются ассинхранные запросы к апи

```ts
import { axios } from '01/axios';

export const deleteDevice = (id: number): Promise<void> =>
  axios.post(`IndividualDevices/${id}/Delete`);
```

### deleteIndividualDeviceService.container.tsx
Контейнер связывает локальную модель сервиса с ее view
```tsx
import { useEvent, useStore } from 'effector-react';
import React from 'react';
import { deleteIndividualDeviceService } from './deleteIndividualDeviceService.models';
import { DeleteIndividualDeviceModal } from './views/DeleteIndividualDeviceModal';

export const DeleteIndividualDeviceModalContainer = () => {
  const visible = useStore(deleteIndividualDeviceService.outputs.$isModalOpen);
  const device = useStore(
    deleteIndividualDeviceService.outputs.$currentIndividualDevice
  );
  const loading = useStore(deleteIndividualDeviceService.outputs.$loading);

  const handleClose = useEvent(
    deleteIndividualDeviceService.inputs.deleteDeviceModalClosed
  );
  const handleDelete = useEvent(
    deleteIndividualDeviceService.inputs.acceptDeleteDevice
  );

  return (
    <DeleteIndividualDeviceModal
      device={device}
      visible={visible}
      loading={loading}
      handleClose={() => handleClose()}
      handleDelete={() => handleDelete()}
    />
  );
};
```

## Использование ttcodegen 

### документация: [ttcodegen](https://www.npmjs.com/package/@pronix/ttcodegen)

### Установка:
```cmd
npm i -g @pronix/ttcodegen
```


