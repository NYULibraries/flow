## `state` Directory to `rsbe phase, step, status` Mapping 

```
|-------------+--------------+--------------+--------|
| State       | rsbe         | rsbe         | rsbe   |
| Directory   | phase        | step         | status |
|-------------+--------------+--------------+--------|
| Queued      | digitization | digitization | queued |
|             |              |              |        |
| Processing  | digitization | digitization | active |
|             |              |              |        |
| QC          | digitization | qc           | active |
|             |              |              |        |
| DoubleCheck | digitization | qc           | error  |
|             |              |              |        |
| ToServer    | upload       | packaging    | queued |
|             |              |              |        |
| UploadOK    | upload       | upload       | done   |
|             |              |              |        |
| UploadFail  | upload       | upload       | error  |
|-------------+--------------+--------------+--------|
```
