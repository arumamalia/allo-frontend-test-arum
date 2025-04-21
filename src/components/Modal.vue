<template>
  <div
    id="errorModal"
    class="modal fade"
    tabindex="-1"
    aria-labelledby="errorModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">
            Error
          </h5>
          <button
            type="button"
            class="btn-close"
            ata-bs-dismiss="modal"
            aria-label="Close"
          />
        </div>
        <div class="modal-body">
          <p>Error fetching rocket list, please try again.</p>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-secondary"
            data-bs-dismiss="modal"
          >
            Retry
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
