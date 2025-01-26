$(document).ready(function () {
    var deleteClientId = 0;

    // When the modal is triggered
    $('#deleteModal').on('show.bs.modal', function (event) {
        var button = $(event.relatedTarget); // Button that triggered the modal
        deleteClientId = button.data('clientid'); // Extract client ID
        var clientName = button.data('clientname'); // Extract client name

        // Update the modal content
        $('#clientName').text(clientName);
    });

    // Confirm delete button click
    $('#confirmDelete').on('click', function () {
        $.ajax({
            url: '/ControllerName/DeleteClientMasterTODB', // Update with your controller's name
            type: 'POST',
            contentType: 'application/json',
            data: JSON.stringify({ clientId: deleteClientId }),
            success: function (response) {
                if (response.success) {
                    alert(response.message || 'Client deleted successfully!');
                    location.reload(); // Refresh the page
                } else {
                    alert(response.message || 'Error deleting client.');
                }
            },
            error: function (xhr, status, error) {
                console.error('Error:', error);
                alert('An unexpected error occurred.');
            }
        });

        // Close the modal
        $('#deleteModal').modal('hide');
    });
});

-------------------------------------------------------------------

@model IEnumerable<dynamic>

<table class="table">
    <thead>
        <tr>
            <th>ClientID</th>
            <th>FirstName</th>
            <th>LastName</th>
            <th>Country</th>
            <th>City</th>
            <th>Phone</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var client in Model)
        {
            <tr>
                <td>@client.ClientID</td>
                <td>@client.FirstName</td>
                <td>@client.LastName</td>
                <td>@client.Country</td>
                <td>@client.City</td>
                <td>@client.Phone</td>
                <td>
                    <!-- Trigger Delete Modal -->
                    <button type="button" class="btn btn-danger" data-bs-toggle="modal" data-bs-target="#deleteModal"
                        data-clientid="@client.ClientID" data-clientname="@client.FirstName @client.LastName">
                        Delete
                    </button>
                </td>
            </tr>
        }
    </tbody>
</table>

<!-- Include the Delete Modal Partial -->
@Html.Partial("_DeleteModal")

<!-- Include External JS -->
<script src="~/js/deleteClient.js"></script>

------------------------------------------------------------------------------

[HttpPost]
public JsonResult DeleteClientMasterTODB(int clientId)
{
    try
    {
        clsmasterdelete deleteOperation = new clsmasterdelete();
        var response = deleteOperation.ExecuteDelete(clientId);

        if (response.StatusCode == 1) // Assuming 1 means success
        {
            return Json(new { success = true, message = response.StatusMessage });
        }
        else
        {
            return Json(new { success = false, message = response.StatusMessage });
        }
    }
    catch (Exception ex)
    {
        return Json(new { success = false, message = "An error occurred while deleting the client: " + ex.Message });
    }
}

----------------------------------------------------------------------------------------------
modal
<div class="modal fade" id="deleteModal" tabindex="-1" aria-labelledby="deleteModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" id="deleteModalLabel">Confirm Delete</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body">
                Are you sure you want to delete <strong id="clientName"></strong>?
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>
                <button type="button" class="btn btn-danger" id="confirmDelete">Delete</button>
            </div>
        </div>
    </div>
</div>
d
