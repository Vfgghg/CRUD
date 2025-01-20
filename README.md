# CRUD
delete
[20/01, 12:45 am] Koyal🎀🎀: public ResponseModel ExecuteDelete(int clientId)
{
    ResponseModel response = new ResponseModel();

    try
    {
        using (SqlConnection conn = new SqlConnection(_connectionString))
        {
            conn.Open();

            using (SqlTransaction transaction = conn.BeginTransaction())
            {
                using (SqlCommand cmd = new SqlCommand("SP_delete_Master", conn, transaction))
                {
                    cmd.CommandType = CommandType.StoredProcedure;

                    // Input parameter
                    cmd.Parameters.AddWithValue("@ClientID", clientId);

                    using (SqlDataReader reader = cmd.ExecuteReader())
                    {
                        if (reader.HasRows)
                        {
                            while (reader.Read())
                            {
                                // Treat both StatusCode and StatusMessage as strings
                                response.StatusCode = reader["StatusCode"].ToString();
                                response.StatusMessage = reader["StatusMessage"].ToString();
                            }
                        }
                    }
                }

                // Commit transaction
                transaction.Commit();
            }
        }
    }
    catch (Exception ex)
    {
        // In case of an error, return a failure response
        response.StatusCode = "500";  // Return as string
        response.StatusMessage = $"Transaction failed: {ex.Message}";
    }

    return response;
}
[20/01, 1:10 am] Koyal🎀🎀: [HttpPost]
public ActionResult DeleteClientmasterTODB(int clientId)
{
    clsmasterdelete deleteOperation = new clsmasterdelete();

    // Execute delete operation and get the response
    ResponseModel response = deleteOperation.ExecuteDelete(clientId);

    // Set ViewBag message
    if (response.StatusCode == "200") // Assuming "200" indicates success
    {
        ViewBag.Message = response.StatusMessage;
        ViewBag.MessageType = "success";
    }
    else
    {
        ViewBag.Message = response.StatusMessage;
        ViewBag.MessageType = "error";
    }

    // Return the updated view with the message
    var model = GetClientList(); // Replace with your method to fetch the updated client list
    return View("Index", model);
}
[20/01, 1:10 am] Koyal🎀🎀: @model IEnumerable<dynamic>

<div>
    @if (ViewBag.Message != null)
    {
        <div class="alert @(ViewBag.MessageType == "success" ? "alert-success" : "alert-danger")">
            @ViewBag.Message
        </div>
    }
</div>

<table class="table table-bordered">
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
                    <form action="@Url.Action("DeleteClientmasterTODB", "YourControllerName")" method="post" style="display:inline;">
                        <input type="hidden" name="clientId" value="@client.ClientID" />
                        <button type="submit" class="btn btn-danger">Delete</button>
                    </form>
                </td>
            </tr>
        }
    </tbody>
</table>

_____________________
edit

using System.Data;
using System.Data.SqlClient;
using System.Configuration;

public class clsmasteredit
{
    private string _connectionString;

    public clsmasteredit()
    {
        _connectionString = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;
    }

    // Method to fetch client details by ClientID
    public dynamic GetClientById(int clientId)
    {
        dynamic client = null;
        SqlConnection conn = new SqlConnection(_connectionString); // Initialize the connection explicitly

        try
        {
            SqlCommand cmd = new SqlCommand("SP_GET_CLIENTDETAILS", conn); // Create a stored procedure for fetching client details
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.Parameters.AddWithValue("@ClientID", clientId);

            conn.Open(); // Explicitly open the connection
            SqlDataReader reader = cmd.ExecuteReader();

            if (reader.HasRows) // Check if there are any rows returned
            {
                while (reader.Read()) // Loop through the rows
                {
                    client = new
                    {
                        ClientID = Convert.ToInt32(reader["ClientID"]),
                        FirstName = reader["FirstName"].ToString(),
                        LastName = reader["LastName"].ToString(),
                        Country = reader["Country"].ToString(),
                        City = reader["City"].ToString(),
                        Phone = reader["Phone"].ToString()
                    };
                }
            }

            reader.Close(); // Explicitly close the reader
        }
        catch (Exception ex)
        {
            throw new Exception("An error occurred while fetching the client details.", ex);
        }
        finally
        {
            conn.Close(); // Explicitly close the connection
            conn.Dispose(); // Explicitly release the connection resource
        }

        return client;
    }

    // Method to update client details
    public int ExecuteEdit(int clientId, string firstName, string lastName, string country, string city, string phone)
    {
        int statusCode = 0;
        SqlConnection conn = new SqlConnection(_connectionString); // Initialize the connection explicitly

        try
        {
            SqlCommand cmd = new SqlCommand("SP_EDIT_CLIENTMASTER", conn);
            cmd.CommandType = CommandType.StoredProcedure;

            // Input parameters
            cmd.Parameters.AddWithValue("@ClientID", clientId);
            cmd.Parameters.AddWithValue("@FirstName", firstName);
            cmd.Parameters.AddWithValue("@LastName", lastName);
            cmd.Parameters.AddWithValue("@Country", country);
            cmd.Parameters.AddWithValue("@City", city);
            cmd.Parameters.AddWithValue("@Phone", phone);

            conn.Open(); // Explicitly open the connection

            // Execute the stored procedure and return the number of rows affected
            int rowsAffected = cmd.ExecuteNonQuery();

            // You can handle the rowsAffected if needed or check your SP for status code
            if (rowsAffected > 0)
            {
                statusCode = 1; // Assuming 1 indicates success
            }
            else
            {
                statusCode = 0; // Failure case
            }
        }
        catch (Exception ex)
        {
            throw new Exception("An error occurred while updating the client details.", ex);
        }
        finally
        {
            conn.Close(); // Explicitly close the connection
            conn.Dispose(); // Explicitly release the connection resource
        }

        return statusCode;
    }
}
